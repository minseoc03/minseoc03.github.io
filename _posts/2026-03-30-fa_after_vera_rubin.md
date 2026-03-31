---
layout: single
title: "What Happens to FlashAttention When Vera Rubin Ships?"
description: "FlashAttention-4 solved Blackwell's asymmetric scaling. Vera Rubin will introduce several new asymmetries — all at once."
date: 2026-03-30 21:26:00
categories: [mlsys, hardware]
author_profile: false
show_excerpts: false
sidebar:
  nav: "mlsys"
---
In the [previous post](https://minseoc03.github.io/mlsys/hardware/flash_attention_4/), we broke down how FlashAttention-4 addresses the fundamental hardware imbalance on Blackwell: tensor cores doubled to 2.25 PFLOPS for BF16, but SMEM bandwidth and MUFU throughput stayed flat. The result was a careful co-design of algorithms and kernels — polynomial exp2 emulation, conditional rescaling, 2-CTA backward passes, TMEM-aware pipelining — all targeting the specific bottlenecks that emerged when matmul got too fast for everything else.

Now, NVIDIA's next-generation **Vera Rubin** platform is entering full production in H2 2026. The Rubin GPU packs 224 SMs (up from 148), HBM4 at 22 TB/s (2.8× Blackwell), and up to 50 PFLOPS of NVFP4 inference. Meanwhile, the Rubin CPX — a separate chip designed for massive-context processing (roughly analogous to the prefill phase in LLM serving) — delivers up to 30 PFLOPS of NVFP4 compute with 128 GB of GDDR7.

The question is: what happens to the attention bottleneck map? And what would a hypothetical "FlashAttention-5" need to look like?

---

## The Asymmetry Gets Worse — And Multiplies

FA4's insight was clean: one asymmetry (tensor cores 2× faster, everything else flat) → identify the shifted bottlenecks → attack each one. On Rubin, the picture is messier because multiple asymmetries stack on top of each other.

### The Numbers We Know

| | Blackwell (B200) | Rubin (R200) | Scaling | Source |
|---|---|---|---|---|
| NVFP4 tensor core (dense) | 10 PFLOPS | 35 PFLOPS | **3.5×** | Official |
| BF16 tensor core | ~2.25 PFLOPS | ~3.6 PFLOPS | **~1.6×** | *Estimated*¹ |
| HBM bandwidth | 8 TB/s | 22 TB/s | **2.8×** | Official |
| SM count | 148 | 224 | **1.5×** | Official |
| NVLink bandwidth/GPU | 1.8 TB/s | 3.6 TB/s | **2×** | Official |
| HBM capacity | 192 GB | 288 GB | **1.5×** | Official |
| SMEM bandwidth/SM | 128 B/cycle | ? | **?** | Not disclosed |
| MUFU throughput/SM | 16 ops/cycle | ? | **?** | Not disclosed |

*¹ BF16 ~3.6 PFLOPS is a back-of-the-envelope estimate based on SM count scaling (224/148 × 2.25 ≈ 3.4) and likely clock speed improvements, not an official NVIDIA specification. NVIDIA's public Rubin disclosures emphasize NVFP4 and FP8 performance.*

The critical unknowns are SMEM bandwidth per SM and MUFU throughput per SM. NVIDIA hasn't disclosed these for Rubin. But the pattern from Hopper → Blackwell is instructive: tensor core throughput doubled per SM, SMEM bandwidth stayed at 128 B/cycle, MUFU stayed at 16 ops/cycle. If this trend continues — even partially — the relative gap between matmul and non-matmul only widens further.

### BF16: A Modest Shift

For BF16 workloads, Rubin's tensor core scaling is relatively modest (~1.6×). If SMEM and MUFU scale similarly or slightly less, the bottleneck profile stays roughly similar to Blackwell. FA4's existing techniques — exp2 emulation, conditional rescaling, 2-CTA SMEM reduction — should port well with minor retuning. The 224 SMs (vs 148) mean more parallelism, which helps with occupancy but doesn't change the per-SM roofline.

This is the easy case.

### FP4: Where Things Get Interesting

FP4 is where Rubin's design intent is most visible. At 3.5× the dense NVFP4 throughput of Blackwell, and with a new Transformer Engine featuring adaptive compression that can push effective throughput toward 50 PFLOPS, the tensor cores are leaving everything else behind at an unprecedented rate.

Let's sketch an FA4-style roofline analysis for a hypothetical NVFP4 attention forward pass. Under a mixed-precision regime, portions of the QK^T and PV matmuls could exploit NVFP4 tensor cores (with FP32 accumulators and scaling metadata), but the softmax — max, subtract, exponentiate, sum, normalize — typically relies on higher precision (often FP32) for numerical stability. You can't meaningfully compute `exp2(x)` at 4-bit precision. This creates a widening gap:

- **MMA time could drop by up to ~3.5×** (if NVFP4 tensor cores are fully utilized for QK^T/PV).
- **Exponential time stays the same** (MUFU is precision-agnostic for transcendentals — it always operates in FP32).
- **SMEM traffic changes modestly** (NVFP4 operands are smaller, but accumulation is still FP32, and P likely needs FP8 or BF16 precision for the PV matmul).

On Blackwell with BF16, the roofline showed MMA and exponential roughly tied (~1024 cycles each for 128³ tiles). With FP4 on Rubin, if MMA cycles drop by 3.5× but exponential stays flat, the exponential unit would dominate by a factor of **~3.5×**. The softmax wouldn't just be a co-bottleneck — it would become the *overwhelming* bottleneck.

This means FA4's current 10-25% FMA emulation split would be wildly insufficient. A Rubin-optimized FP4 attention kernel might need:

1. **50-75% FMA emulation** — or whatever ratio matches the new MMA-to-MUFU throughput gap.
2. **Higher-degree polynomials** — FP4's quantization error is enormous (~6% relative error), so a degree-3 polynomial is still fine. But if intermediate precision increases (e.g., FP8 softmax outputs), degree-4 or degree-5 might be needed.
3. **Alternative approximation strategies** — at some point, polynomial emulation on FMA units hits diminishing returns due to register pressure and latency. Lookup table–based approaches, piecewise linear approximations, or even dedicated softmax-aware hardware (which Rubin's Transformer Engine might provide) could be the next step.

---

## The Adaptive Compression Wild Card

Rubin's most architecturally novel feature for attention is the **3rd-generation Transformer Engine with adaptive compression**. Unlike Blackwell's structured 2:4 sparsity (which required exactly half of values to be zero, and which almost nobody used in practice), Rubin's adaptive compression dynamically detects and skips zeros in the data stream without forcing values to zero.

*What follows is speculative — NVIDIA has not disclosed how adaptive compression interacts with fused attention kernels at the SM level. But the architectural implications are worth exploring.*

One possible implication for attention: the attention matrix P = softmax(QK^T) is naturally sparse in many practical settings — especially with causal masking, local attention windows, or after the first few tokens where most attention weights are near zero. If the Transformer Engine can skip near-zero P values during the PV matmul, the effective FLOPS for PV could drop without any algorithmic change.

But exploiting this — if it is exposed to software at the tensor-input boundary — would likely require the FA kernel to:

1. **Produce P in a format the Transformer Engine can compress.** This might mean specific memory layouts, quantization schemes, or metadata that the hardware compression can consume.
2. **Coordinate compression with tiling.** If compression ratios vary across tiles (dense attention regions vs. sparse regions), the pipeline timing becomes data-dependent — potentially disrupting the carefully balanced ping-pong schedules that FA4 uses.
3. **Potentially rethink the softmax-to-MMA boundary.** Currently, P is computed, stored to TMEM, and consumed by the PV MMA. If the compression engine operates between these stages, there might be an additional pipeline step — or it might be entirely transparent if the hardware handles it on the MMA input path.

These are open design questions, not known hardware contracts. But they illustrate the kind of hardware-algorithm co-design that FA4 demonstrated is essential.

---

## The Backward Pass: SMEM Pressure Intensifies

FA4's backward pass analysis showed that shared memory traffic exceeds MMA compute by ~30% in the 1-CTA case, reduced to ~5% with 2-CTA mode. On Rubin, if tensor cores get faster while per-SM SMEM bandwidth stays flat (as it did from Hopper to Blackwell — though this is not yet confirmed for Rubin), the backward pass becomes even more SMEM-dominated.

FA4's 2-CTA approach halves operand B traffic by having each CTA stage only half of B. A natural extension *would be* larger cooperative MMA patterns — e.g., 4-CTA or cluster-wide operand sharing — though whether Rubin's hardware supports this is unknown at time of writing. Blackwell's 2-CTA mode was a new capability that didn't exist on Hopper; Rubin may similarly introduce new cooperative primitives.

Similarly, if TMEM capacity increases on Rubin (not yet disclosed), more intermediate results could stay in TMEM, reducing SMEM round-trips. These are possibilities to watch for in the Rubin architecture documentation, not confirmed features.

The dQ atomic reduction problem also scales: with more SMs (224 vs 148), more CTAs are writing to the same dQ tiles, increasing atomic contention. The deterministic backward pass's semaphore-based serialization becomes more expensive as the CTA count grows.

---

## Rubin CPX: A Completely Different Attention Problem

The Rubin CPX is perhaps the most interesting challenge for FlashAttention. Designed for massive-context processing (the compute-heavy "context phase" roughly analogous to prefill in LLM serving), it has a radically different compute-to-bandwidth ratio compared to the R200:

| | R200 | Rubin CPX | Source |
|---|---|---|---|
| NVFP4 compute | 35 PFLOPS (dense) | up to 30 PFLOPS | Official |
| Memory | 288 GB HBM4 | 128 GB GDDR7 | Official |
| HBM/memory bandwidth | 22 TB/s | *Not officially disclosed per-chip*² | — |
| **Compute/BW ratio** | ~1,590 FLOPS/byte | **Substantially higher** | — |

*² NVIDIA discloses 1.7 PB/s memory bandwidth for the full NVL144 CPX rack (144 GPUs), but per-chip GDDR7 bandwidth has not been officially specified. Third-party estimates place it around 2 TB/s, which would imply ~10,000-15,000 FLOPS/byte — roughly 6-10× more compute-heavy than the R200. The exact ratio matters less than the directional conclusion: CPX is far more compute-dense relative to its memory bandwidth.*

This extreme ratio means:

### FlashAttention's IO-Awareness Becomes Even More Critical

The entire point of FlashAttention was to avoid materializing the N×N attention matrix in HBM. On CPX, with relatively limited memory bandwidth compared to its compute, even modest memory traffic is costly. The tiling strategy must be even more aggressive — larger tiles to maximize compute per byte loaded, potentially at the cost of more TMEM/register pressure.

### Recomputation Becomes Free(er)

On CPX, compute is abundant relative to bandwidth. FlashAttention already recomputes S = QK^T in the backward pass rather than storing it. On CPX, you could afford to recompute even more aggressively — for example, recomputing partial softmax results rather than storing statistics, or computing QK^T multiple times with different tile decompositions to optimize memory access patterns.

### The Kernel Architecture Might Invert

On the R200, FA4 tries to keep the tensor cores busy and hide SMEM/MUFU latency. On CPX, the tensor cores have so much throughput relative to memory bandwidth that the kernel might be **memory-bound even with FlashAttention's tiling**. In this regime, the optimization target flips: instead of "keep tensor cores busy," it becomes "minimize every byte of memory traffic, even if tensor cores idle." This might mean:

- **Longer inner loops** (process more KV blocks before writing back) to amortize the cost of loading Q.
- **Fused QKV loading** — loading Q, K, V in a single TMA operation rather than separate stages.
- **KV cache compression in-flight** — if the Transformer Engine can decompress KV cache on the fly during the MMA, the effective memory bandwidth for loading K and V increases.

This is essentially a **different kernel for the same algorithm** — same FlashAttention math, but with the pipeline optimized for a completely different hardware profile.

---

## Million-Token Context: Rack-Scale Attention

Vera Rubin NVL72 connects 72 GPUs with NVLink 6 at 3.6 TB/s per GPU. With 288 GB HBM per GPU, the total memory pool is 20.7 TB — enough for million-token contexts with large KV caches.

But million-token attention means N = 1,000,000, and even with FlashAttention's linear memory, the *compute* scales as O(N²). At some point, the KV cache for a single head doesn't fit on one GPU, and attention becomes a **distributed problem**.

Current approaches (like Ring Attention and sequence parallelism) tile the sequence across GPUs and overlap communication with computation. But the FA4-style roofline analysis hasn't been applied to the distributed case. The relevant "feeds and speeds" expand to include:

- **NVLink bandwidth** (3.6 TB/s) for moving KV blocks between GPUs
- **NVLink latency** for synchronizing softmax statistics across sequence partitions
- **Load balancing** across GPUs when causal masking makes some partitions much heavier than others

An FA5 for Rubin might need to co-design the **intra-SM pipeline** (tile-level, as FA4 does) with the **inter-GPU communication schedule** (sequence-level). The LPT scheduling that FA4 applies within a single GPU could extend to scheduling across GPUs — placing the heaviest sequence partitions on GPUs that finish their local work first.

---

## The Framework Advantage

Here's where FA4's most practical contribution — the CuTe-DSL framework — pays compound dividends. Adapting attention kernels to Rubin's new hardware requires rapid experimentation:

- Tuning the FMA/MUFU emulation ratio for each precision (FP4, FP8, BF16).
- Exploring new tile sizes and pipeline stages for the different compute/bandwidth ratios (R200 vs CPX).
- Integrating with the new Transformer Engine's adaptive compression APIs.
- Prototyping cluster-wide MMA patterns (4-CTA or beyond).

In the FA3 era, each such experiment required modifying C++ templates and waiting 55 seconds per compile. With CuTe-DSL's 2.5-second JIT compilation, a researcher can iterate 20× faster. This isn't just a convenience — it fundamentally changes what's practical to explore.

The modular design also helps: the block-sparse, masking, and scheduling primitives are orthogonal, so adapting to Rubin's new features (adaptive compression, larger clusters) can be done by adding new primitives without rewriting the core softmax or MMA logic.

---

## Summary: The Bottleneck Multiplication Table

| Bottleneck | Blackwell (FA4) | Rubin R200 (BF16) | Rubin R200 (FP4) | Rubin CPX |
|---|---|---|---|---|
| **Tensor core** | Co-bottleneck | Moderate | Fast | Very fast |
| **MUFU (exp)** | Co-bottleneck | Similar | **Dominant** | **Dominant** |
| **SMEM bandwidth** | Bwd bottleneck | Worse | Much worse | Less relevant |
| **HBM bandwidth** | Not bottleneck | Not bottleneck | Not bottleneck | **Dominant** |
| **Cross-GPU comm** | N/A | Long context | Long context | Depends on disaggregated serving topology |

FA4 addressed a single clean asymmetry. On Rubin, the attention kernel faces **multiple simultaneous asymmetries** that vary by precision (FP4 vs BF16), chip variant (R200 vs CPX), and scale (single GPU vs rack). The methodology — roofline analysis per resource, targeted algorithmic mitigation — remains sound. But the number of configurations to optimize explodes.

The good news: FA4's CuTe-DSL framework, modular primitives, and tuning infrastructure were designed for exactly this kind of rapid adaptation. The question isn't whether the attention algorithm can keep up with the hardware. It's whether the iteration speed of the framework can keep up with the proliferation of hardware targets.

If the Hopper → Blackwell transition is any guide, the answer is that the first team to have working, optimized kernels on new silicon wins — and then everyone else (including NVIDIA's own cuDNN team) converges on the same techniques. FA4 demonstrated this pattern. On Rubin, the race starts again.
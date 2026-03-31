---
layout: single
title: "FlashAttention-4: When Tensor Cores Got Too Fast for Everything Else"
description: "A deep dive into how FlashAttention-4 co-designs algorithms and kernels to tame asymmetric hardware scaling on NVIDIA Blackwell GPUs"
date: 2026-03-30 20:43:00
categories: [mlsys, hardware]
author_profile: false
show_excerpts: false
sidebar:
  nav: "mlsys"
---

Every generation of GPU hardware promises faster matrix multiplication. Blackwell delivers on that promise — doubling tensor core throughput to 2.25 PFLOPS for BF16 compared to Hopper's 1 PFLOP. But here's the twist: **everything else stayed roughly the same.** Shared memory bandwidth? Unchanged. Exponential unit throughput? Unchanged. The result is a fundamentally lopsided machine where the matmul engine is screaming ahead while the rest of the chip struggles to keep up.

FlashAttention-4, from Tri Dao's group at Princeton (with collaborators at Meta, Colfax Research, and NVIDIA), is the response to this new reality. Rather than porting FlashAttention-3's Hopper-optimized kernels forward — which would either leave performance on the table or simply not compile — FA4 rebuilds from scratch around Blackwell's actual bottlenecks. The results: up to 1613 TFLOPS/s on B200 (71% utilization), 1.3× faster than cuDNN 9.13, and 2.7× faster than Triton.

Let's break down what changed and why.

---

## The Problem: Asymmetric Hardware Scaling

On Hopper (H100), the performance balance between tensor cores, shared memory, and special function units was tight enough that a well-pipelined kernel could keep most resources busy. On Blackwell (B200), that balance is shattered.

Here are the numbers per SM per clock cycle:

| Resource | Hopper (H100) | Blackwell (B200) | Scaling |
|---|---|---|---|
| Tensor core (BF16 MMA) | 4,096 ops | 8,192 ops | **2×** |
| Shared memory bandwidth | 128 bytes | 128 bytes | **1×** |
| Exponential unit (MUFU) | 16 ops | 16 ops | **1×** |

The implication is stark. On Hopper, the attention kernel was roughly compute-bound — the bottleneck was tensor core throughput. On Blackwell, the bottleneck has shifted to **shared memory traffic** and **exponential unit throughput**. The roofline analysis in the paper confirms this: for a typical 128³ tile configuration, SMEM and exponential each take ~1024 cycles while MMA also takes ~1024 cycles. They're neck-and-neck. But for larger tiles (256 × 128²), MMA and exponential both take 2048 cycles — meaning the kernel now has *two* co-dominant bottlenecks that are no longer matmul.

The forward pass bottleneck is the exponential unit. The backward pass bottleneck is shared memory bandwidth, exceeding MMA compute time by ~30%.

---

## What's New in Blackwell Hardware

Before diving into the algorithmic fixes, it's worth understanding the new hardware primitives that FA4 exploits.

### Tensor Memory (TMEM)

Blackwell introduces **256 KB of tensor memory per SM** — a new level in the memory hierarchy specifically coupled to the tensor cores. On Hopper, MMA results were written back to registers, which created enormous register pressure and limited tile sizes. On Blackwell, MMA outputs go directly to TMEM, asynchronously. This is a game-changer for two reasons:

1. **Register pressure relief.** Intermediate accumulator values no longer consume the limited 256-register-per-thread budget.
2. **Decoupled execution.** Since MMA writes to TMEM are fully async, the tensor cores don't block on register writeback, enabling better overlap with other operations.

TMEM is allocated in 32-column (16 KB) granules and requires explicit programmer management — it's not a transparent cache.

### Larger Tile Sizes

Blackwell MMA instructions operate on **128 × 128** tiles (doubled from Hopper's 64 × 128). This means fewer instructions per output tile, but also that each instruction touches more shared memory — amplifying the SMEM bandwidth bottleneck.

### 2-CTA MMA Mode

Blackwell supports a cooperative mode where two CTAs within a cluster jointly execute a single MMA. The accumulator is partitioned along the M dimension, and — critically — each CTA only stages **half** of operand B in its shared memory while the hardware consumes the combined B tile. This effectively halves redundant SMEM traffic for operand B.

---

## Fix #1: Redesigned Pipeline for MMA & Softmax Overlap

### The Core Idea

Since softmax computation (exponentials, max, normalization) is now a co-bottleneck with MMA, the pipeline must be designed so that **while one tile's MMA is running, another tile's softmax is being computed** — and vice versa. FA4 uses a **ping-pong schedule** (similar in spirit to FA3) where two output tiles are computed per thread block, but the implementation is fundamentally different due to Blackwell's TMEM and larger tiles.

### FA3 vs FA4: Thread Assignment

On Hopper (FA3), each row of an accumulator tile was distributed across **4 threads in an interleaved pattern**. Computing a row-wise operation like softmax required **inter-warp shuffles** — threads had to exchange partial results to reconstruct full rows before computing the row max and row sum. This shuffle overhead was a tax on every softmax step.

On Blackwell (FA4), the 128×128 tile and TMEM layout allow **each thread to own an entire row of 128 elements**. Two warpgroups of 128 threads each handle softmax, with each thread processing one complete row. This eliminates inter-warp shuffles entirely. The softmax flow per warpgroup becomes:

```
1. Load entire row from TMEM into registers
2. Compute row max
3. Subtract max, exponentiate, convert to BF16
4. Compute row sum
5. Store P tile to TMEM
```

No shuffles. No partial reductions across threads. Clean and fast.

### Decoupled Rescaling via Correction Warpgroup

In FA3, the online softmax rescaling step — where previous output tiles are multiplied by `exp(m_old - m_new)` to account for updated row maxima — happened inline within the softmax warpgroup. This put rescaling squarely on the **critical path**.

In FA4, because intermediate results are communicated via TMEM rather than registers, the rescaling is offloaded to a separate **"correction" warpgroup**. The softmax warpgroups compute P and hand off rescale statistics through TMEM; the correction warpgroup applies the rescaling independently. This takes rescaling off the critical path of the main softmax + MMA pipeline.

The overall warpgroup assignment per thread block:

| Warpgroup | Role |
|---|---|
| WG0 | Tensor core driver + TMA (data movement) |
| WG1 | Softmax (tile H — "high" Q tile) |
| WG2 | Softmax (tile L — "low" Q tile) |
| WG3 | Correction (rescaling) |

The two softmax warpgroups are explicitly synchronized so their exponential-heavy critical sections don't overlap, ensuring they don't fight for the same MUFU resource.

### TMEM Partitioning Strategy

With 256 KB of TMEM per SM, the allocation must be carefully planned:

**Mandatory:** Two output tiles (O^H, O^L) for the ping-pong schedule. At head dimension 128 with FP32 accumulators, each output tile is 128 × 128 × 4 bytes = 64 KB, so two tiles consume 128 KB — exactly half.

**Remaining 128 KB** can store either:
- **Option A:** 1 S tile (FP32, 64 KB) + 2 P tiles (BF16, 32 KB each) = 128 KB
- **Option B:** 2 S tiles (FP32, 64 KB each) = 128 KB, with P overwriting S in-place

FA4 chooses **Option B**. Why?

- **Immediate compute start.** Two S tiles can be computed back-to-back at pipeline startup, filling the pipeline faster.
- **In-place conversion.** Once S is consumed by softmax and converted to P (which is BF16, half the size), P overwrites S. This is safe because S is no longer needed after softmax.
- **Statistics storage.** The leftover space (since P is smaller than S) is used to store rescale statistics for the correction warpgroup.

### Register Pressure Management

Here's where things get tight. Each SM has 65,536 registers shared across all threads. With 4 warpgroups × 128 threads = 512 threads per thread block, that's 128 registers per thread.

But each softmax thread needs:
- **128 registers** to hold a full row of input (128 BF16 values, packed 2 per register = 64, but with FP32 intermediates during computation, it's 128)
- **~64 registers** for output staging
- Additional registers for temporaries, loop variables, coefficients

This exceeds the budget. FA4's solution: **staged P storage.** The first 3/4 of P is computed, stored to TMEM (which triggers the corresponding MMA), and registers are freed. Then the remaining 1/4 is computed separately. This trades a small amount of instruction overhead for staying within register limits.

---

## Fix #2: Software-Emulated Exponential Function

### The Bottleneck

The MUFU (Multi-Function Unit) handles transcendental functions like `exp2(x)`. At 16 ops/cycle/SM, it's 512× slower than the tensor cores (8192 ops/cycle/SM). For attention, every element of the M × N score matrix needs an exponential evaluation during softmax. This makes MUFU a hard bottleneck.

### The Trick: Polynomial Approximation on FMA Units

FA4 implements `2^x` in software using the classical Cody-Waite range reduction + polynomial approximation:

```
2^x = 2^⌊x⌋ × 2^(x - ⌊x⌋)
```

The integer part `2^⌊x⌋` is computed via **bit manipulation** of the IEEE 754 exponent field — essentially a shift-and-add on integer ALU.

The fractional part `2^frac` (where frac ∈ [0, 1)) is approximated by a low-degree polynomial evaluated with **FMA (fused multiply-add) instructions**:

```
2^frac ≈ p0 + p1·frac + p2·frac² + p3·frac³
```

evaluated via Horner's method:

```
result = ((p3·frac + p2)·frac + p1)·frac + p0
```

Each step is a single FMA. The key insight: **FMA units can run in parallel with MUFU**, effectively doubling (or more) the exponential throughput by splitting work across both functional units.

### Why Not Emulate Everything?

Full emulation has costs:
- **More registers** for polynomial coefficients and intermediates
- **Higher latency** per evaluation than a single MUFU instruction
- **Register bandwidth** consumption that can cause spills

So FA4 takes a hybrid approach: **10–25% of exponentials are computed via FMA emulation**, with the rest going through hardware MUFU. The exact split is tuned empirically based on the MMA-to-exponential throughput ratio for each tile configuration.

### Accuracy

The degree-3 polynomial has a max relative error of ~8.8 × 10⁻⁵ in FP32. Sounds bad? It doesn't matter. After rounding to BF16, the quantization error (~3.9 × 10⁻³) completely dominates the polynomial approximation error. The degree-3 polynomial matches hardware MUFU to within 1 BF16 ULP on 99% of inputs. For attention, where softmax outputs are consumed at BF16 precision, this is more than sufficient.

---

## Fix #3: Conditional Softmax Rescaling

### The Problem

Standard online softmax (as used in all FlashAttention versions) maintains running statistics:

```
m_j = max(m_{j-1}, rowmax(S_j))
ℓ_j = exp(m_{j-1} - m_j) · ℓ_{j-1} + rowsum(exp(S_j - m_j))
O_j = exp(m_{j-1} - m_j) · O_{j-1} + exp(S_j - m_j) · V_j
```

The rescaling term `exp(m_{j-1} - m_j) · O_{j-1}` is a full vector multiplication that happens **every iteration**, even when the max barely changes.

### The Fix

FA4 introduces a threshold τ (set to log₂(256) = 8.0):

```
if m_j - m_{j-1} > τ:
    # Full rescale (standard path)
    O_j = exp(m_{j-1} - m_j) · O_{j-1} + exp(S_j - m_j) · V_j
else:
    # Skip rescale, use old max
    O_j = O_{j-1} + exp(S_j - m_{j-1}) · V_j
```

When the new max doesn't exceed the old max by more than τ, the rescaling is skipped entirely. The key insight: **the final normalization step `Output = O_final / ℓ_final` corrects any accumulated drift.** As long as intermediate values don't overflow (which the threshold of 256× prevents), the final result is exact.

In practice, rescaling is needed only in the first few iterations when the running max is still being established. Once it stabilizes, most iterations skip rescaling entirely — saving a vector multiply per iteration. To avoid warp divergence, the rescaling decision is made per-warp: if **any** thread in the warp needs rescaling, all threads rescale.

---

## Fix #4: 2-CTA Backward Pass

The backward pass is where shared memory pressure is most severe. With five MMA operations (vs. two in the forward pass), SMEM traffic exceeds MMA compute by ~30% in the 1-CTA configuration.

### How 2-CTA Helps

In 2-CTA MMA mode, a CTA pair cooperatively executes each MMA with M = 256 (each CTA holds half the M dimension). The critical benefit: **each CTA only stages half of operand B** in shared memory, while the hardware reads the combined B tile from both CTAs. This roughly halves SMEM traffic for operand B across the five backward GEMMs.

### The dQ Problem

There's a catch. The dQ computation accumulates along the KV sequence dimension (the outer loop), and its reduction axis is N — which is naturally split across the CTA pair. Each CTA still needs the **full reduction** for its rows.

FA4 solves this using **Distributed Shared Memory (DSMEM)** to exchange half of the dS tile between the two CTAs in the same cluster. After the exchange, each CTA holds an (M/2 × 2N) slice of dS, enabling a CTA-pair UMMA with doubled reduction dimension. The per-CTA dQ MMA becomes:

```
dQ tile shape: (M/2, 2N) × (2N, d) → (M/2, d)
```

This restructuring also **halves the number of global atomic reductions** for dQ, since each CTA writes only half the dQ tile. Atomics are expensive and introduce nondeterminism, so cutting them in half is a double win.

### Deterministic Mode

For reproducible training (critical for RL applications), FA4 provides a deterministic execution mode using semaphore-based serialization of global reductions. The performance overhead is minimized through careful CTA scheduling:
- Batches processed as the outermost dimension
- Heads swizzled within L2 cache capacity
- For causal masking: KV blocks launched in descending order, query blocks in ascending order from the diagonal, dQ reductions ordered by descending query block index ("shortest-processing-time-first")

This gets the deterministic backward pass to ~75% the speed of the nondeterministic 1-CTA version.

---

## Fix #5: LPT Scheduling

Load imbalance is inherent in attention — causal masking means tiles near the diagonal do more work than tiles far from it; variable-length batches have different sequence lengths.

FA4 applies **Longest-Processing-Time-First (LPT) scheduling**, a classical result from parallel processing theory. For causal masking, the naive grid order processes tiles from shortest to longest (because the grid iterates Q blocks in order, and early Q blocks have fewer valid KV blocks). LPT reverses this, processing the heaviest tiles first to minimize makespan.

The scheduling also considers L2 cache locality: heads are divided into sections that fit in L2 cache, and the scheduler traverses heads-per-section → mblocks (reversed) → sections → batches. For GQA/MQA, all query heads per KV head are traversed before varying over mblocks.

Empirical gains: **4–8% FLOPS improvement for MHA, 7–14% for MQA** on H200 (this optimization is architecture-agnostic and also benefits FA3 on Hopper).

---

## The Framework: CuTe-DSL

Perhaps the most practically impactful contribution for the broader ecosystem: FA4 is implemented **entirely in CuTe-DSL embedded in Python** — no C++ template metaprogramming whatsoever.

### Compile Time

| | Forward | Backward |
|---|---|---|
| FA3 (C++ templates) | 55s | 45s |
| FA4 (CuTe-DSL) | 2.5s | 1.4s |
| **Speedup** | **22×** | **32×** |

And FA3 required precompiling **hundreds of kernels** for different attention variants. FA4's JIT compilation means you compile only what you need, when you need it.

### Why It Matters

CuTe-DSL is isomorphic to CUTLASS C++ — same programming model, same expressivity, same access to low-level GPU primitives — but with Python's meta-programming ergonomics and JIT compilation. When CuTe-DSL's APIs don't cover something, raw PTX is available as an escape hatch.

The practical impact is already visible: developers have built **FlexAttention and block-sparse attention variants** on top of FA4's framework without modifying core code. The barrier to entry drops from "years of C++ template metaprogramming expertise" to "a few months of GPU programming experience."

The design philosophy is modular: block-sparse patterns, masking strategies, variable-length handling, and work scheduling are all orthogonal, composable primitives. New attention variants get all existing optimizations for free.

---

## Results

On B200 with BF16, head dimension 128:

- **Forward pass:** 1.1–1.3× faster than cuDNN 9.13, 2.1–2.7× faster than Triton. Peak: **~1600 TFLOPS** (71% of theoretical max).
- **Backward pass:** Consistent speedups across sequence lengths for both causal and non-causal.
- **Deterministic backward:** Up to 75% the speed of the non-deterministic 1-CTA backward — a practical option for RL training that demands reproducibility.

Notably, since FA4's release, the cuDNN team has incorporated many of FA4's techniques into cuDNN 9.14+, converging to similar performance. This is perhaps the strongest validation of the algorithmic contributions.

---

## Key Takeaway

FlashAttention-4 is a case study in what happens when you take hardware asymmetry seriously. The tensor cores got faster, so the bottleneck moved. Rather than hoping the bottleneck would go away, the authors identified exactly where the cycles were being spent (roofline analysis with concrete cycle counts for every resource), then attacked each bottleneck with a targeted fix:

- SMEM too slow → 2-CTA MMA to halve operand B traffic
- Exponential unit too slow → Polynomial emulation on FMA units
- Rescaling wastes cycles → Conditional rescaling with deferred correction
- Pipeline bubbles → Ping-pong schedule with TMEM-based decoupling
- C++ compile times too slow → CuTe-DSL in Python

The code is open source at [github.com/Dao-AILab/flash-attention](https://github.com/Dao-AILab/flash-attention/tree/main/flash_attn/cute).

*The B300/GB300 will double MUFU throughput to 32 ops/cycle/SM. When that happens, the bottleneck map will shift again. The cycle continues.*


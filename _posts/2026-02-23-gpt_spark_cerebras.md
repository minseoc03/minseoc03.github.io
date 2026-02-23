---
layout: single
title: "Spark, Cerebras, and the Future of Low-Latency AI Inference"
description: "Recent thoughts about Cerebras and Spark model"
date: 2026-02-23 01:10:54
categories: [mlsys, hardware]
author_profile: false
show_excerpts: false
sidebar:
  nav: "mlsys"
---

On February 12th, OpenAI released GPT-5.3-Codex-Spark, the first model designed for real-time coding and inferenced using the Cerebras WSE-3.

This release was notable because it marked the first time OpenAI publicly deployed a model powered by Cerebras hardware, following their reported multi-billion-dollar partnership. I had been anticipating this launch, as it seemed like a potential inflection point — one that could challenge the long-standing GPU dominance in AI infrastructure and open the door for alternative chip architectures to play a serious role.

OpenAI described Spark as an initial step toward expanding Cerebras usage across more frontier models as they scale their WSE-based datacenter capacity. Below are benchmark results for GPT-5.3-Codex-Spark.

|![swebench.png](\assets\images\hardware\swebench.png)|
|:--:| 
| *SWE-Bench Pro Benchmark* |

|![terminalbench.png](\assets\images\hardware\terminalbench.png)|
|:--:| 
| *Terminal-Bench 2.0 Benchmark* |

The benchmarks show that the model does not necessarily achieve the highest accuracy across all metrics. However, that is not the primary objective. The defining characteristic is speed. The model delivers ultra-low latency responses, which is precisely where Cerebras hardware is designed to excel.

Importantly, OpenAI did not rely solely on Cerebras hardware to achieve these gains. They also made substantial changes to their inference infrastructure. Instead of the original statement, we can summarize their approach as follows: OpenAI restructured their streaming pipeline to reduce client-server communication overhead, redesigned parts of the inference stack, and improved session initialization so the first token appears more quickly. By introducing persistent WebSocket connections and optimizing the Responses API, they significantly reduced per-request overhead, per-token processing latency, and time-to-first-token. These improvements are now enabled by default for Codex-Spark and are expected to become standard across other models.

This led me to a central question. 
> *"If OpenAI is actively integrating Cerebras hardware and planning broader adoption, does this signal a shift in chip market dominance? Could GPUs be replaced?"*

The answer appears to be no.

OpenAI has clearly stated that GPUs remain foundational to their training and inference pipelines. GPUs continue to provide the most cost-effective token generation at scale and remain the backbone of general-purpose AI workloads. Cerebras hardware, on the other hand, is optimized for specialized demand — particularly scenarios that require extremely low latency. Rather than replacing GPUs, OpenAI’s strategy appears to combine both architectures, leveraging each where it performs best.

After reading the release notes, I became more curious about what makes the Cerebras chip fundamentally different and what kind of software stack supports it.

## Cerebras Chip Architecture
OpenAI explicitly stated that Spark runs on the WSE-3 chip.

Unlike traditional accelerators that cut a silicon wafer into many smaller dies, Cerebras keeps the entire 300mm wafer intact and turns it into a single massive processor. Instead of connecting many chips together via high-speed interconnects, WSE-3 eliminates chip-to-chip communication by implementing a large-scale on-wafer mesh network.

|![WSE Chip.png](\assets\images\hardware\wsechip.png)|
|:--:| 
| *WSE-3 Chip* |

This design raises two natural questions. 
- Why did the industry historically avoid wafer-scale processors? 
- If we serve models on such hardware, how must the software stack change?

#### Why Chips Were Traditionally Cut

The answer lies in economics and physics.

Semiconductor manufacturing inevitably produces defects. If defect density is denoted as D and chip area as A, yield approximately decreases exponentially as area increases. As chips become larger, the probability that a defect affects the die increases significantly. Smaller dies improve overall manufacturing yield because a defect only invalidates a small portion of the wafer rather than the entire structure. For decades, yield optimization directly translated into economic viability.

Lithography constraints also played a role. Photolithography equipment has a maximum reticle size, meaning only a limited region can be patterned in one exposure. Larger chips require stitching techniques that historically introduced reliability and alignment challenges. These limitations reinforced the industry’s preference for smaller dies.

Packaging and thermal management further favored modular designs. Smaller chips are easier to cool, test, and replace. A wafer-scale processor requires specialized power delivery systems, advanced cooling solutions, and routing mechanisms capable of bypassing defective regions. Cerebras invested heavily in defect-aware routing and custom cooling to make wafer-scale computing feasible.

In short, traditional design philosophy prioritized yield, modularity, and replaceability. Wafer-scale computing only became viable when technological advances and AI workload characteristics justified the trade-off.

#### What WSE-3 Changes Architecturally

Traditional GPU systems assume a distributed architecture. Multiple accelerators are connected via NVLink, PCIe, or other networking fabrics. Each device has limited memory bandwidth, and large models must be partitioned carefully. Communication primitives such as AllReduce become central to scaling performance.

Wafer-scale computing inverts this assumption. Instead of distributing computation across multiple chips, the system behaves as a single spatially distributed processor. The interconnect exists directly on the wafer, dramatically reducing communication distance and latency. Memory is distributed across on-chip SRAM rather than relying primarily on external HBM.

This is not merely a hardware upgrade. It alters the abstraction model upon which modern ML systems software has been built.

#### How the Model Serving Pipeline Changes

In a conventional GPU serving stack, models are defined in frameworks such as PyTorch or JAX, compiled through intermediate representations like ONNX, optimized by tools such as TensorRT, and lowered into CUDA kernels. Runtime systems manage tensor parallelism, pipeline parallelism, and KV cache sharding, while NCCL handles collective communication. The serving infrastructure is fundamentally distributed.

With WSE-3, the structure shifts.

If a model fits on a single wafer, tensor parallelism may become less necessary. Rather than partitioning parameters across multiple devices, the optimization problem becomes one of spatial placement across hundreds of thousands of processing elements. The compiler’s focus shifts from kernel fusion and warp scheduling to global graph mapping and routing efficiency.

Memory management also changes. In GPU systems, developers carefully manage HBM, L2 cache, shared memory, and registers. KV cache eviction and sharding policies are designed around distributed memory constraints. In a wafer-scale architecture with distributed SRAM, memory locality becomes a graph-placement problem rather than a multi-device synchronization problem.

Serving strategy shifts as well. GPU clusters often prioritize throughput via dynamic batching and aggressive utilization strategies. Wafer-scale systems, with enormous on-chip bandwidth and minimal inter-device communication overhead, may enable different latency-throughput trade-offs. Some distributed coordination complexity disappears, but placement and routing complexity increases.

#### What Tooling Exists Today

Cerebras provides a software development kit designed specifically for its hardware. The Cerebras Software Language allows low-level programming of wafer processing elements, and simulator environments enable development without direct hardware access. There are also model repositories adapted for Cerebras systems and Python SDKs for inference through API endpoints.

However, the ecosystem remains more vertically integrated than CUDA-based development. Hardware access is typically restricted, and much of the software stack is tightly coupled to Cerebras infrastructure. While tooling exists, it is not as broadly accessible or mature as mainstream GPU ecosystems.

For those interested in exploring the SDK and CSL in more detail, Cerebras provides documentation and tutorials at the following link:

https://sdk.cerebras.net/computing-with-cerebras

#### Final Thoughts
The release of GPT-5.3-Codex-Spark does not signal the end of GPU dominance. Instead, it represents a diversification of AI hardware strategies.

GPUs remain foundational for large-scale training and cost-efficient inference. Cerebras hardware, particularly WSE-3, appears optimized for extreme low-latency workloads. OpenAI’s approach suggests a hybrid infrastructure model, where different chips are deployed based on workload characteristics rather than ideological preference.

The more interesting question is not whether Cerebras will replace GPUs, but how heterogeneous hardware architectures will reshape compiler design, serving infrastructure, and system-level optimization in the coming years.


---
layout: single
title: "LLM Serving 101: Prefill, Decode, Batching, and the Systems Behind Large Language Models"
description: "Recent thoughts about MXFP4 and GPT-OSS"
date: 2026-01-16 21:10:54
categories: [mlsys, inference]
author_profile: false
show_excerpts: false
sidebar:
  nav: "mlsys"
---
> **Prerequisites**
This post assumes that the reader is already familiar with the Transformer architecture, including self-attention, autoregressive decoding, and the role of key–value (KV) caches.
If you are new to Transformers, it is recommended to review the basics before reading, as this post focuses on inference-time systems rather than model fundamentals.

Large Language Models (LLMs) are usually discussed from a model-centric perspective: parameter counts, attention mechanisms, training tricks, or dataset scale. However, once a model is trained, a very different problem dominates real-world usage:
>  
> **How do we serve LLMs efficiently, reliably, and at scale?**
> 

LLM Serving has little to do with model accuracy. Instead, it is governed by:
- latency
- throughput (tokens per second)
- GPU memory efficiency
- cost per generated token

This post is a foundational but deep overview of modern LLM serving systems, covering:
- how inference is structured,
- why prefill and decode behave so differently,
- where the real bottlenecks come from,
- and how modern systems mitigate them.

### 1. What is LLM Serving?
At its core, LLM serving is the **inference-time system** that sits between users and GPUs.
A typical serving pipeline looks like this:
1. User sends a request
2. The request enters a queue
3. A scheduler decides when and how it runs
4. A batcher decides which other requests to group it with
5. The GPU executes inference
6. Tokens are streamed back to the user

What makes this difficult is that:
* prompts are variable-length
* users arrive asynchronously
* GPUs are optimized for large, regular workloads

All of these suggest that serving is fundamentally a **systems problem**, not a modeling problem.

### 2. Inference Into Two Stages : Prefill vs. Decode
A critical insight in LLM serving is that **inference is split into two different stages**.
#### 2.1 Prefill Stage
Prefill processes the entire prompt in one forward pass and builds the KV cache looking at entire input tokens.

Characteristics:
- Input: all prompt tokens (length = $T$)
- Attention: full self-attention ($QK^T \in \mathbb{R}^{T \times T}$)
- Output:
    - KV cache (keys and values for every layer)
    - hidden states for the first token generation

Since prefill can look at entire tokens, it brings a **high GPU utilization** due to large matrix-matrix mulitplications and high arithmetic intensity. However, it is also where **latency explodes for long prompts**.

#### 2.2 Decode Stage
Decode stage generates new tokens autoregressively, one token at a time. It means each token generation depends on all previous tokens.
$$x_t \sim P(x_t | x_1, \dots, x_{t-1})$$
It suggests that this stage forces sequential computing, underutilizing GPUs due to lack of parallelism.

Characteristic:
- Input: a single token per step
- Attention: $\mathbb{R}^{1 \times T}$ (query attends to KV cache)
- Repeated until generation ends (reachs termination token)

Decode is fundamentally different to prefill because:
- sequential dependency across time
- small kernels
- memory-bound execution

| Stage   | Attention Shape | Bottleneck           | Parallelism |
|---------|------------------|----------------------|-------------|
| Prefill | T × T            | Compute + memory     | High        |
| Decode  | 1 × T            | Memory bandwidth     | Very low    |

This split explains why optimizing inference is not a single problem.

### 3. Why Prefill Dominates Latency
#### 3.1 Quadratic Attention Cost
Self-attention computes:
$$\text{Attention}(Q,K,V) = \text{softmax}(QK^T)V$$
For prefill:
- $Q \in \mathbb{R}^{T \times d}$
- $K \in \mathbb{R}^{T \times d}$
- $QK^T \in \mathbb{R}^{T \times T}$

This means doubling prompt length quadraples attention work (= $O(T^2)$) and long-context users dominate wall-clock latency, leading to tail latency by a small fraction of long prompts.

#### 3.2 KV Cache Creation Cost
Prefill must materialize keys and values for **every token** and for **every transformer layer**. This results in heavy GPU memory writes, pressure on HBM capacity, and reduced concurrency. This tells that KV cache is not just a decode bottleneck—it is created in prefill.

#### 3.3 Scheduling Side Effects
Prefill kernels are large, long-running, and non-preemptive. Due to these properties, a single long-prompt prefill can monopolize the GPU, block decode requests, and inflate latency. This causes delaying entire service due to few long prompt users. This is why prefill is both a **compute bottleneck** and a **scheduling bottleneck**.

### 4. Why Decode Is Still Slow
Despite much less computation, decode introduces its own challenges.
#### 4.1 Autoregressive Dependency
Each token depends on all previous tokens as mentioned above. This dependency prevents parallelization along the time dimension and enforces strict sequential execution.
#### 4.2 Memory Bandwith Bottleneck
Decode attention reads the entire KV cache from HBM and performs relatively little computation. This incurs a large data exchange between SM and HBM, leading to memory bound. Since FLOPs are cheap and HBM bandwith dominates, GPU utilization drops sharply.
#### 4.3 Kernel Launch Overhead
Decode consists of many small kernels and repeated thousands of times per request. Launch overhead and poor fusion become visible performance costs.

### 5. Batching: The Central Lever of Serving Performance
Batching is a technique of combining multiple requests and dealing it with a single GPU execution. Without batching, as LLM inference is a large matmul computation, handling single request each time makes most SMs idle, increasing latency and costs per token.
#### 5.1 Why Naive Batching Fails
Prompts vary widely in length.
```txt
Request A: 8000 tokens
Request B: 200 tokens
Request C: 100 tokens
```
Batching forces padding to the maximum length. Because attention is $O(T^2)$, padding causes real computation, not just wasted memory, so large batches can reduce throughput. This is a length variance problem.
#### 5.2 Modern Batching Strategies
**Dynamic Batching**
- Collect requests within a short time window
- Trade latency for throughput

**Length-Aware Batching**
- Group requests with similar prompt lengths
- Trade latency for throughput

**Token-Level (Continuous) Batching**
- Batch tokens, not sequences
- Mix prefill and decode steps
- Prevent long requests from blocking short ones

Token-level batching is the foundation of modern serving engines such as vLLM.

### 6. Moden Solutions to Core Bottlenecks
#### 6.1 FlashAttention (Prefill Optimization)
FlashAttention algorithm avoids materializing $T \times T$ attention matrix and keeps computation in SRAM, resulting in reducing memory traffic from $O(T^2)$ to $O(T)$. It does not remove quadratic computation, but makes it practical.
#### 6.2 Paged KV Cache
- Store KV cache in page-sized blocks
- Reduce fragmentation
- Increase Concurrency
This directly improves decode stability.
#### 6.3 Chunked Prefill
Instead of processing long prompts in one large kernel:
- split prefill into chunks
- interleave with decode steps

Benefits:
- lower peak latency
- better scheduling fairness
- reduced tail latency
#### 6.4 Quantization (Decode-Focused)
Lower-precision formats:
- reduce memory bandwidth
- benefit decode more than prefill
- improve throughput at scale
#### 6.5 Speculative Decoding
Idea:
- a small model drafts multiple tokens
- a large model verifies them

Effect:
- fewer autoregressive steps
- lower perceived latency

### 7. Putting All Together
| Component  | Primary Bottleneck     | Key Techniques                     |
|------------|------------------------|------------------------------------|
| Prefill    | $O(T^2)$ attention        | FlashAttention, chunking           |
| Decode     | Memory bandwidth       | Token batching, quantization       |
| Scheduling | GPU monopolization     | Priority, chunked prefill          |
| Memory     | KV cache               | Paging, compression                |

### 8. Conclusion
> **LLM serving is not about running a model.**
> **It is about orchestrating computation, memory, and time.**

This post focused on building a **high-level mental model** of LLM serving—how inference is structured, where the real bottlenecks come from, and why serving is fundamentally a systems problem.

The individual optimization techniques discussed here, such as FlashAttention, KV cache management, token-level batching, chunked prefill, and decode-aware scheduling, are each deep topics on their own.
They will be explored in more detail in future posts.

In practice, these techniques are not applied in isolation.
They are already integrated into modern **inference engines and serving platforms**, such as:
- **vLLM** for efficient GPU-side execution,
- **Ray Serve** for flexible and scalable request handling,
- **KServe** on top of Kubernetes for production deployment and orchestration.

LLM serving is not about a single optimization, but about how many optimizations work together as a system.

This post provides the map; the next posts will dive into the individual components that make modern LLM inference engines work at scale.


---
layout: single
title:  "Intro to CUDA"
date:   2025-01-12 21:10:54 
categories: [mlsys, cuda]
author_profile: false
sidebar:
  nav: "mlsys"
---

## Introduction to CUDA

### GPU vs. CPU

- The GPU is a chipset specialized for parallel processing.  
- A GPU has far more cores than a CPU, but the performance of a single GPU core is much weaker compared to a CPU core.  
- However, by executing a large number of cores in parallel, the GPU significantly accelerates simple computations.  
- In contrast, the CPU excels in single-core performance and demonstrates high efficiency in handling complex computations with fewer cores.  

### Concepts of CUDA

**Model Structure**

- **Thread**: A thread is the smallest unit of computation in parallel processing. Each thread executes the same instruction but works with different data. This approach is called SIMT (Single Instruction Multiple Thread).  
- **Block**: A block is a group of threads that share the same memory (also referred to as *Shared Memory*).  
- **Grid**: A grid is a collection of blocks required to perform a computation.  

**Memory Structure**

- **Global Memory**: The largest memory unit accessible by all threads, but relatively slow in access speed.  
- **Shared Memory**: Memory allocated per block, accessible by all threads within that block.  
- **Local Memory**: Memory allocated individually to each thread.  
- **Constant & Texture Memory**: Used to store read-only data and cache data.  

**Kernel Functions**

- Every kernel executed on the GPU must include the `__global__` tag.  
- To reference blocks and threads, the syntax `<<< >>>` is used.  

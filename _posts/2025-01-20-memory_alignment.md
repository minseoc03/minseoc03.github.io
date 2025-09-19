---
layout: single
title:  "Memory Alignment and Coalescing"
date:   2025-01-20 21:10:54 
categories: [mlsys, cuda]
author_profile: false
sidebar:
  nav: "mlsys"
---

## Memory Optimization

### Memory Alignment

- Memory alignment means storing data at addresses that are multiples of the data size.  
- For example, since an `int` has a size of 4 bytes, storing it at an address that is a multiple of 4 is more efficient.  
- Why is this the case?  
    - If data is stored at aligned addresses, large chunks of data can be fetched in a single memory access, fully utilizing memory bandwidth.  
    - If data is unaligned, multiple memory accesses may be required, leading to inefficiency.  

### Memory Coalescing

- The reason memory alignment improves efficiency is due to a concept called **memory coalescing**.  
- You might wonder: how do we actually enforce alignment at multiples of the data size? The answer is memory coalescing.  
- Memory coalescing refers to storing data in **consecutive addresses**, for example:  
    - Thread 0 → A[0]  
    - Thread 1 → A[1]  
    - …  
    - Thread 31 → A[31]  
- In this case (assuming an `int` array), memory addresses are assigned as:  
    - 0x00 → A[0]  
    - 0x04 → A[1]  
    - 0x08 → A[2]  
- With this layout, the 32 threads in a warp access consecutive memory addresses.  
- As a result, only a **single memory transaction** is needed to fetch all the required data, maximizing bandwidth and ensuring highly efficient execution.  

---
layout: single
title:  "Bank Conflict"
date:   2025-01-25 21:10:54 
categories: [mlsys, cuda]
author_profile: false
sidebar:
  nav: "mlsys"
---

## Bank in Shared Memory

### What is a Bank?

- A **bank** is a component of shared memory. In other words, multiple banks together form shared memory.  
- Most modern GPUs organize shared memory into **32 banks**, and each bank can handle 32 bits (4 bytes) of data per clock cycle.  
    - This means that in one clock cycle, a bank can process one `int` or one `float`.  

### Bank Conflict

- Banks can significantly reduce GPU performance if not used carefully.  
- If two or more threads access the **same bank** at the same time, the accesses are serialized, which slows execution. This is called a **bank conflict**.  

### Optimization

- To optimize performance, shared memory access patterns should avoid bank conflicts.  
- Cases where no conflicts occur:  
    - If all threads in a warp access the **same address**, CUDA treats it as a single operation. This is called **broadcast**, and no conflicts occur.  
    - If shared memory stores `int` or `float` data and threads access it in a **coalesced** manner, then each thread handles 4 bytes, and each thread maps to a different bank. This ensures conflict-free access.  

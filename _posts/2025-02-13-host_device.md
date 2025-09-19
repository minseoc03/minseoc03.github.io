---
layout: single
title:  "Host-Device Synchronization"
date:   2025-02-13 21:10:54 
categories: [mlsys, cuda]
author_profile: false
sidebar:
  nav: "mlsys"
---

# Host-Device Synchronization

- As discussed earlier, the **host** refers to the CPU, and the **device** refers to the GPU.  
- By default, CUDA launches kernels **asynchronously**. This means that the CPU does not wait for the GPU to finish and immediately continues executing subsequent instructions.  
- In such cases, operations may become unsynchronized, leading to incorrect results.  
- To avoid these issues, we need to learn how to explicitly synchronize between the CPU and GPU.  

## Overview

- CUDA kernels are executed asynchronously by default. This implies:  
    - The CPU does not wait for the GPU computation to finish.  
    - Memory access may occur before the kernel finishes, potentially copying only partial data.  
    - Explicit synchronization is required to prevent such issues.  
- Example of an error caused by asynchronous execution:  

```cpp
vectorAddKernel<<<numBlocks, numThreads>>>(d_A, d_B, d_C, N);
cudaMemcpy(h_C, d_C, size, cudaMemcpyDeviceToHost); // Incorrect: GPU may not be done
```

- Corrected version with synchronization:  

```cpp
vectorAddKernel<<<numBlocks, numThreads>>>(d_A, d_B, d_C, N);
cudaDeviceSynchronize(); // Wait until GPU computation is done
cudaMemcpy(h_C, d_C, size, cudaMemcpyDeviceToHost);
```

## Understanding Host-Device Synchronization

### Problems with Asynchronous Execution

- The CPU may proceed before the GPU has completed its computation.  
    - This can result in incomplete or incorrect data being returned to the host.  
- Multiple kernels may execute asynchronously.  
    - Without explicit ordering, kernels may overlap, leading to incorrect sequencing.  
- Memory access may occur before GPU computations are complete.  
    - Results may be invalid or corrupted.  

### Solutions

- Explicit synchronization ensures clear ordering between host and device operations.  

## Synchronization Methods

| Method | Description | When to Use |
| --- | --- | --- |
| `cudaDeviceSynchronize()` | Blocks the CPU until all previously launched CUDA operations are finished. | When host operations depend on the completion of all CUDA operations. |
| `cudaMemcpy()` | Memory transfer functions inherently synchronize. | When memory access is required only after computations are finished. |
| `cudaEventSynchronize(event)` | Synchronizes only with a specific event. | When only a specific operation among multiple operations requires synchronization. |

- While frequent use of `cudaDeviceSynchronize()` guarantees correct execution order, overuse unnecessarily stalls CPU execution, leading to performance degradation.  

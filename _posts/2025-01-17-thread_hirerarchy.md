---
layout: single
title:  "Thread Hierarchy"
date:   2025-01-17 21:10:54 
categories: [mlsys, cuda]
author_profile: false
sidebar:
  nav: "mlsys"
---

## CUDA Thread Hierarchy

- CUDA organizes threads in a hierarchical structure to efficiently manage parallel execution. This hierarchy is divided into:
    - Grid
    - Block
    - Warp

### Grids and Blocks

- **Grid**  
    - A grid is created whenever a kernel is launched, and it is composed of blocks.  
    - When launching a kernel, you must specify the number of blocks inside the grid.  
    - Grids can be 1D, 2D, or 3D, depending on the computational purpose.  
- **Block**  
    - A block is a group of threads. Threads inside a block can collaborate through shared memory and can synchronize their execution.  
    - Like grids, blocks can be 1D, 2D, or 3D depending on the application.  
- **Key Characteristics**  
    - A block is the unit of thread scheduling.  
    - Threads inside a block can communicate and synchronize with each other.  
    - Blocks are independent of one another and cannot directly communicate.  

### Warps

- **Warp**  
    - A warp is a group of 32 threads that execute the same instruction simultaneously on an SM.  
    - Warps are the smallest execution unit in GPU computation.  
- **Key Characteristics**  
    - Under SIMT execution, all threads in a warp follow the same instruction.  
    - Excessive branching (e.g., conditionals) can cause *divergence*, reducing performance.  

## Thread Indexing

- Thread indexing is essential for mapping data correctly to each thread.  

### Global Indexing

- Global indexing assigns a unique index to every thread in the entire grid, allowing data mapping to individual threads.  
- Example:  

```cpp
int globalIdx = blockIdx.x * blockDim.x + threadIdx.x;
```

- Variables:  
    - `blockIdx.x` = index of the block within the grid  
    - `blockDim.x` = number of threads per block  
    - `threadIdx.x` = index of the thread within its block  
- Why this calculation? Suppose there are 3 blocks, each with 4 threads:  
    - Block 0 → Threads 0 ~ 3  
    - Block 1 → Threads 0 ~ 3  
    - Block 2 → Threads 0 ~ 3  
    - Using the formula, the global indices become:  

    | **blockIdx.x** | **threadIdx.x** | **globalIdx Calculation** | **Result** |
    | --- | --- | --- | --- |
    | 0 | 0 | 0 * 4 + 0 | 0 |
    | 0 | 1 | 0 * 4 + 1 | 1 |
    | 0 | 2 | 0 * 4 + 2 | 2 |
    | 1 | 0 | 1 * 4 + 0 | 4 |
    | 1 | 1 | 1 * 4 + 1 | 5 |
    | 2 | 3 | 2 * 4 + 3 | 11 |

### Block Indexing

- Blocks can also be indexed to determine their position.  

```cpp
int blockIdx = blockIdx.x;

// For 2D or 3D configurations:
int blockIdx_x = blockIdx.x;
int blockIdx_y = blockIdx.y;
int blockIdx_z = blockIdx.z;
```

### Example

- Each thread can be mapped to different data with code like this:  

```cpp
__global__ void exampleKernel(float *A, int N) {
    int globalIdx = blockIdx.x * blockDim.x + threadIdx.x;
    if (globalIdx < N) {
        A[globalIdx] = globalIdx;
    }
}
```

## Specifying Grid and Block Sizes

- Choosing appropriate grid and block sizes is critical for efficient GPU utilization and optimal performance.  

### Factors to Consider

- **Number of Threads**  
    - The number of threads should exceed the number of data elements.  
- **Hardware Specs**  
    - GPUs have limits on threads per block, usually up to 1,024.  
    - Register count and shared memory size affect how many blocks can run simultaneously.  
- **Occupancy**  
    - The ratio between the maximum warps that can be assigned to an SM and the number of active warps.  
- **Memory Access Patterns**  
    - Data should be mapped to consecutive memory addresses for faster access.  

### Optimal Settings

- **Powers of 2**  
    - Computer systems generally favor powers of 2. Block sizes are best set as powers of 2 (e.g., 128, 256, 512).  
- **Multiples of 32**  
    - Since a warp consists of 32 threads, block sizes should be multiples of 32 for full warp utilization.  
- **Hardware Constraints**  
    - Block size must not exceed hardware limits.  

### Example

```cpp
int threadsPerBlock = 256; // 512, 1024 also possible
int blocksPerGrid = (N + threadsPerBlock - 1) / threadsPerBlock;
```

- In most cases, block sizes are set like this.  

## Block Synchronization

- As mentioned earlier, threads inside a block can synchronize with one another.  
- This is especially important when using shared memory.  

### Syncthreads

- This function ensures that all threads in a block complete their current instruction before moving forward.  
- Often used during shared memory operations to avoid crashes.  

```cpp
__global__ void synchronizedKernel(float *A, float *B, float *C, int N) {
    __shared__ float sharedData[256];
    int idx = blockIdx.x * blockDim.x + threadIdx.x;
    
    if (idx < N) {
        sharedData[threadIdx.x] = A[idx] * 2.0f;
        __syncthreads();
        C[idx] = sharedData[threadIdx.x] + B[idx];
    }
}
```

- In this example, `__syncthreads()` ensures all threads write to shared memory before proceeding to the next operation.  
- Note: While there are other synchronization methods, synchronization across blocks is not possible since blocks are independent.  

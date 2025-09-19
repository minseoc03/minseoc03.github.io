---
layout: single
title:  "Kernel Configuration"
date:   2025-02-09 21:10:54 
categories: [mlsys, cuda]
author_profile: false
sidebar:
  nav: "mlsys"
---
## Kernel Configuration

- As mentioned earlier, CUDA performance varies widely depending on hardware capabilities.  
- When programming CUDA, you should choose the number of threads per block with hardware limits in mind.  
- You should consider **occupancy**: high occupancy generally indicates good utilization of the hardware, but excessively high occupancy is not always best.  
- Register pressure and shared memory limits can throttle performance and actually slow execution.  

## Example

- To compare configurations, we perform vector addition while varying the number of threads per block.

```cpp
// kernelTuning.cu
#include <cuda_runtime.h>
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

// Declaration of the vector addition kernel.
__global__ void vectorAddKernel(const float *A, const float *B, float *C, int N) {
    int idx = blockDim.x * blockIdx.x + threadIdx.x;
    if (idx < N) {
        C[idx] = A[idx] + B[idx];
    }
}

int main() {
    // Vector length.
    int N = 1 << 20;  // 1M elements
    size_t size = N * sizeof(float);

    // Allocate host memory for input vectors and output vector.
    float *h_A = (float*)malloc(size);
    float *h_B = (float*)malloc(size);
    float *h_C = (float*)malloc(size);

    // Initialize host arrays with random values.
    srand(time(NULL));
    for (int i = 0; i < N; i++) {
        h_A[i] = (float)(rand() % 100) / 10.0f;
        h_B[i] = (float)(rand() % 100) / 10.0f;
    }

    // Allocate device memory for vectors.
    float *d_A, *d_B, *d_C;
    cudaMalloc((void**)&d_A, size);
    cudaMalloc((void**)&d_B, size);
    cudaMalloc((void**)&d_C, size);

    // Copy input data from host to device.
    cudaMemcpy(d_A, h_A, size, cudaMemcpyHostToDevice);
    cudaMemcpy(d_B, h_B, size, cudaMemcpyHostToDevice);

    // Define a range of block sizes to test (e.g., multiples of warp size 32).
    int blockSizes[] = {32, 64, 128, 256, 512};
    int numBlockSizes = sizeof(blockSizes) / sizeof(blockSizes[0]);
    float timeTaken;

    // Create CUDA events for timing.
    cudaEvent_t start, stop;
    cudaEventCreate(&start);
    cudaEventCreate(&stop);

    // Loop through the different block sizes.
    for (int i = 0; i < numBlockSizes; i++) {
        int threadsPerBlock = blockSizes[i];
        int blocksPerGrid = (N + threadsPerBlock - 1) / threadsPerBlock;

        // Record the start event.
        cudaEventRecord(start);

        // Launch the kernel with the current configuration.
        vectorAddKernel<<<blocksPerGrid, threadsPerBlock>>>(d_A, d_B, d_C, N);

        // Record the stop event.
        cudaEventRecord(stop);
        cudaEventSynchronize(stop);

        // Calculate the elapsed time.
        cudaEventElapsedTime(&timeTaken, start, stop);
        printf("Block Size: %d, Blocks Per Grid: %d, Time Taken: %f ms\n", threadsPerBlock, blocksPerGrid, timeTaken);
    }

    // Optionally, copy the result back to host for correctness verification.
    cudaMemcpy(h_C, d_C, size, cudaMemcpyDeviceToHost);

    // Free device memory.
    cudaFree(d_A);
    cudaFree(d_B);
    cudaFree(d_C);

    // Free host memory.
    free(h_A);
    free(h_B);
    free(h_C);

    // Destroy CUDA events.
    cudaEventDestroy(start);
    cudaEventDestroy(stop);

    return 0;
}
```

- Sample output:

```text
Block Size: 32, Blocks Per Grid: 32768, Time Taken: 165.995926 ms
Block Size: 64, Blocks Per Grid: 16384, Time Taken: 208.894424 ms
Block Size: 128, Blocks Per Grid: 8192, Time Taken: 289.929138 ms
Block Size: 256, Blocks Per Grid: 4096, Time Taken: 294.194855 ms
Block Size: 512, Blocks Per Grid: 2048, Time Taken: 207.752121 ms
```

- In this experiment, a block size of **32** (one warp per block) yields the fastest runtime.  
- However, the kernel is extremely simple and occupancy is quite low, so overall GPU resources are likely underutilized.  

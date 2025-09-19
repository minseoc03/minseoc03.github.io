---
layout: single
title:  "Advanced Atomic Operations"
date:   2025-02-05 21:10:54 
categories: [mlsys, cuda]
author_profile: false
sidebar:
  nav: "mlsys"
---
## Advanced Atomic Operations

### Importance of Atomic Operations

- Although explained in the previous note, let’s restate the concept.  
- Atomic operations exist to prevent what is commonly known in parallel computing as **race conditions**.  
- A race condition occurs when two or more threads attempt to access the same memory location simultaneously. In CUDA, this leads to serialized execution, slower performance, and potentially incorrect updates if values overlap.  
- To prevent this, CUDA provides atomic operations, which serialize access safely when multiple threads attempt to update the same variable.  

### atomicCAS()

- CUDA provides basic atomic operations such as `add`, `sub`, `min`, and `max`. However, `atomicCAS` is important because it allows the creation of **custom atomic operations**.  
- `atomicCAS` (compare and swap) compares a value at a memory location with a given value, and if they are equal, it swaps the memory value with a new one.  

```cpp
__device__ int atomicCAS(int* address, int compare, int val);
```

- Structure: if the value at `address` matches `compare`, replace it with `val`.  
- Return value: the old value at `address` (if no swap occurred, the current value is returned).  

### atomicExch()

- Unlike `atomicCAS`, this function **unconditionally swaps** the value at a memory location.  

```cpp
__device__ int atomicExch(int* address, int val);
```

- Structure: directly replaces the value at `address` with `val`.  
- Return value: the old value at `address`.  

### Example: Using atomicCAS to Implement atomicMax for Floats

- CUDA provides `atomicMax()` for integers, but not for floats.  
- Using `atomicCAS`, we can implement a custom version for floats:  

```cpp
// __device__ function to implement atomicMax for floats using atomicCAS.
// CUDA does not provide native atomicMax for float values.
__device__ float atomicMaxFloat(float* address, float val) {
    // Convert float pointer to int pointer for atomicCAS usage
    int* address_as_i = (int*)address;
    int old = *address_as_i, assumed;

    do {
        assumed = old;
        // If another thread updates the value while current thread is working,
        // 'assumed' will not match 'address_as_i'.
        // atomicCAS will then return a different value,
        // triggering the loop to retry computation and avoid race conditions.
        old = atomicCAS(address_as_i, assumed, __float_as_int(fmaxf(val, __int_as_float(assumed))));
    } while (assumed != old);

    return __int_as_float(old);
}
```

- Steps:  
    - Save the current value into `old` and `assumed`.  
    - If another thread modifies the value in between, `assumed` will differ from the memory content, so no swap occurs.  
    - Since the return value of `atomicCAS` will also differ, the `while` loop is triggered, and the computation is retried.  
- This way, race conditions are prevented, and the maximum value in a float array can be safely computed.  

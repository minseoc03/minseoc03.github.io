---
layout: single
title:  "LLVM-Block (OSS)"
date:   2025-05-05 21:10:54 
categories: [mlsys, llvm]
author_profile: false
sidebar:
  nav: "mlsys"
---

### LLVM-BLOCK

- **LLVM-BLOCK** is an open-source tool released by the KC-ML2 research team. Its purpose is to **compare two LLVM IR modules and identify basic blocks that remain unchanged**.
- More concretely, it takes LLVM IR before and after optimization (or two IRs from different builds) and **matches blocks that are preserved**.
- Why is this useful? For example:
  - You may want to know which parts of the code a particular optimization pass actually changed while leaving others intact.
  - Or when comparing IR generated under two different compiler options, LLVM-BLOCK helps you **focus only on the changed blocks** by quickly identifying the unchanged ones.

### How It Works

- LLVM-BLOCK relies on **debug information for block matching**.
- As seen earlier, LLVM IR debug metadata records the original source location of each instruction.
- LLVM-BLOCK compares the **debug info metadata** between `IR_before` (unoptimized) and `IR_after` (optimized) to find blocks corresponding to the same source code.
- Therefore, you must generate IR with the `-g` option to include debug information when using LLVM-BLOCK.
- Without debug info, even identical code may be hard to match due to renamed registers, but with debug metadata, blocks can be identified as equivalent if they map back to the same source.

### Example Use Cases

- **CFG comparison before and after optimization**
  - A common use case is analyzing **how a specific optimization pass changes the CFG**.
  - For instance, you can compare IR at `-O0` versus `-O3` to see which blocks disappear or remain.
  - The ML2 team blog demonstrates this by comparing the CFG of a `DoReplace` function under `-O0` and `-O3` and showing that LLVM-BLOCK identifies the headers of blocks that remain unchanged.

### How to Use

1. **Prepare IR_before**
    - Compile the target program with options like `-O0 -g` to generate LLVM IR with debug info. (`.ll` file)
2. **Prepare IR_after**
    - Apply optimizations to `IR_before` to get a second IR.
    - For example:  
      ```bash
      opt -mem2reg -gvn -S before.ll -o after.ll
      ```  
      Or directly use Clang with `-O2`/`-O3` to emit optimized IR.
3. **Run LLVM-BLOCK**
    - Execute the tool with both IR files:  
      ```bash
      llvm-block before.ll after.ll
      ```  
    - LLVM-BLOCK writes results to `stderr`, so redirect to a file if needed:  
      ```bash
      llvm-block before.ll after.ll 2> output.txt
      ```
4. **Check Results**
    - Output lists the headers of **identical BasicBlocks per function**.
    - For example, in function `foo`, unchanged blocks might be `entry, loop, exit`.
    - You can then focus your manual comparison only on the changed blocks.
    - Optionally, you can also strip debug info (`opt -strip-debug`) and use Graphviz (`opt -dot-cfg`) to visualize CFGs, then cross-check with LLVM-BLOCK’s output.

### Things to Watch Out For

- LLVM-BLOCK is only meaningful if both IRs come from the **same source codebase**.
    - Comparing two completely different programs will yield no useful matches.
    - Normally, you compare IRs produced with different optimization levels or flags.
- LLVM-BLOCK was developed around **LLVM 10**, so it may not handle newer features (e.g., opaque pointers from LLVM 15+) without adjustments.

### Summary

- **LLVM-BLOCK** is an **IR comparison tool** that helps identify **unchanged basic blocks between two IR modules**.
- It’s particularly useful for compiler optimization research and IR-level diffs.
- Since it relies on debug info, always generate IR with `-g`.
- By highlighting the **invariant parts**, it makes it easier to spot and analyze the **changed regions** in complex IR code.

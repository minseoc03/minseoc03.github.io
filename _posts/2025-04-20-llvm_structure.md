---
layout: single
title:  "LLVM Basic Structure"
date:   2025-04-20 21:10:54 
categories: [mlsys, llvm]
author_profile: false
sidebar:
  nav: "mlsys"
---

### Overall Structure of LLVM

![image.png](/assets/images/llvm/image.png)

- LLVM is a **modular compiler framework** designed to separate the traditional compiler pipeline into **Front-end**, **Middle-end**, and **Back-end**, allowing each part to be independently replaced or extended.  
- For example, the C/C++ compiler front-end **Clang** parses C/C++ source code into LLVM **IR** (Intermediate Representation), which is then passed to the LLVM **Optimizer** (middle-end).  
- The Optimizer applies **multiple optimization passes** to transform the IR into a new optimized IR, which is then passed to the back-end to be emitted as **assembly or binary code** for the target machine.  
- Thanks to this structure, LLVM can be used by many different front-ends (e.g., Swift, Rust, Fortran) to translate into a common IR, apply rich optimizations, and then generate code for multiple architectures (x86, ARM, PowerPC, etc.).  

### Compilation Flow

The overall compilation flow of LLVM can be summarized as follows:

- **Front-end stage:** The source code is lexed and parsed into an AST, which is then translated into LLVM **IR**. (e.g., Clang generates IR from C/C++ source code).  
- **Middle-end (Optimizer) stage:** The IR undergoes **various optimization passes** to improve performance. Since IR is hardware-independent, these optimizations are language- and platform-agnostic.  
- **Back-end stage:** The optimized IR is translated into target **machine code**. At this stage, the virtual registers and abstract instructions of the IR are mapped to the actual registers and instructions of the CPU.  

### In Summary

- LLVM IR acts as a **hub** connecting the front-end and back-end. Multiple front-ends can emit LLVM IR, and multiple back-ends can consume the same IR, enabling efficient support for many languages and platforms.  
- Front-end developers only need to translate source code into IR, while back-end developers only need to work with IR. This makes LLVM a powerful framework that provides both **language independence** and **architecture independence**.  

---
layout: single
title:  "LLVM IR Syntax"
date:   2025-04-22 21:10:54 
categories: [mlsys, llvm]
author_profile: false
sidebar:
  nav: "mlsys"
---

### Components and Syntax of IR

- LLVM IR has an assembly-like syntax and is organized at the **module** level.  
- A module can contain multiple **global variables** and **function definitions**, each of which is composed of **basic blocks**.  
- The key components of IR are:  
    - **Module**  
        - Represents a single compiled program unit. It includes target information (target triple), data layout, global variables, and function declarations/definitions.  
        - For example, at the top of an IR file you may see:  
          `target triple = "x86_64-pc-linux-gnu"`  
    - **Global Variable**  
        - Variables/constants used throughout the program, named with `@`.  
        - Example:  
          `@.str = private constant [14 x i8] c"Hello, world\0A\00", align 1`  
          Here, `@.str` is the global constant name, `[14 x i8]` indicates a 14-byte string, and `align 1` specifies alignment.  
    - **Function Declaration and Definition**  
        - Declared with `declare` and defined with `define`.  
        - Example:  
          `declare i32 @printf(ptr, ...)`  
          `define i32 @add(i32 %a, i32 %b) { ... }`  
        - Functions are written as:  
          `define <return type> @<name>(<parameters>) <attributes> { ... }`  

### Example

C function:

```c
int add(int a, int b) {
    int sum = a + b;
    return sum;
}
```

Generated IR (via `clang -O0 -S -emit-llvm code.c -o code.ll`):

```llvm
define i32 @add(i32 %a, i32 %b) {
entry:
  %sum = add i32 %a, %b        ; add a and b, assign to sum
  ret i32 %sum                 ; return sum
}
```

- `define i32 @add(i32 %a, i32 %b)` defines a function `add` that takes two `i32` parameters and returns an `i32`.  
- `%a`, `%b` are **virtual registers** holding argument values.  
- `entry:` is a **basic block label**.  
- `%sum = add i32 %a, %b` is an **add instruction** storing the result in `%sum`.  
- `ret i32 %sum` returns `%sum`.  

### SSA (Static Single Assignment)

- LLVM IR follows **SSA form**, meaning each value is assigned exactly once.  
- Registers (e.g., `%sum`) are unique within a function.  

### Global Identifiers

- Identifiers starting with `@` are **global symbols** (functions, global variables).  
- Types (e.g., `i32`, `i8*`) are explicitly attached to operands and instructions.  
- Example: `add i32 %a, %b` requires both operands to be `i32`.  

### Common IR Instructions

- Arithmetic: `add`, `sub`, `mul`, `udiv`, etc.  
- Comparison: `icmp`, `fcmp` (return `i1` boolean values).  
- Control flow: `br`, `switch`.  
- Phi nodes: `phi` (for SSA merges).  
- Memory: `load`, `store`.  
- Stack/other: `alloca`, `call`, `ret`.  

Example (C: `int x = 10; int y = x + 5;`, compiled with `-O0`):

```llvm
%x = alloca i32, align 4           ; allocate stack space for x
store i32 10, ptr %x, align 4      ; store constant 10 into x
%tmp = load i32, ptr %x, align 4   ; load x into tmp
%y = add i32 %tmp, 5               ; compute y = tmp + 5
```

### In Summary…

- LLVM IR is structured as **Module → Function → Basic Block → Instruction**.  
- Its assembly-like syntax, explicit typing, and SSA form allow precise analysis and optimization during compilation.  

---
layout: single
title: "Compiler Basic Structure"
date: 2025-06-02 21:10:54
categories: [mlsys, compiler]
author_profile: false
sidebar:
  nav: "mlsys"
---
# Compiler Theoretical Structure

```
Source Program
     ↓
Lexical Analyzer  → (Symbol Table)
     ↓ tokens
Syntax Analyzer   → (Symbol Table)
     ↓ AST
Intermediate Code Generation → (Symbol Table)
     ↓ Intermediate Code
Optimization      → (Symbol Table)
     ↓ Intermediate Code
Code Generation   → (Symbol Table)
     ↓ Object Code
Object Program
```

### Explanation

- **Lexical Analyzer**: Converts the source program into a sequence of **tokens**.  
- **Syntax Analyzer**: Builds the **AST (Abstract Syntax Tree)** from tokens.  
- **Intermediate Code Generation**: Produces an **intermediate representation** of the program.  
- **Optimization**: Improves the **intermediate code** for efficiency.  
- **Code Generation**: Translates optimized intermediate code into **object code**.  
- **Object Program**: Final output of the compiler, ready for execution.  
- **Symbol Table**: Central data structure accessed at all stages for storing identifiers, types, and attributes.

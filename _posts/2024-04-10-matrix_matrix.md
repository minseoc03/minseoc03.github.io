---
layout: single
title:  "Matrix derivative respect to Matrix"
date:   2024-04-10 21:03:54 
categories: [ml, math]
author_profile: false
sidebar:
  nav: "ml"
---
## 1-15. 행렬을 행렬로 미분

- 기존의 방식으로는 미분이 불가능
- 행렬을 벡터로 풀은 이후 벡터를 벡터로 미분하는 방식으로 미분을 진행해야함.
    - $vec(\begin{bmatrix}x_{11} & x_{12} \\ x_{21} & x_{22}\end{bmatrix}) = \begin{bmatrix}x_{11} & x_{12}&x_{13}&x_{14}\end{bmatrix}$
    - $F(X) = AX \\ d \space vec(F) = d\space vec(X)\cdot\frac{\partial \space vec(F)}{\partial \space vec^T(X)}$
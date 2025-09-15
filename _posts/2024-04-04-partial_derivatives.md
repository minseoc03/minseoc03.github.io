---
layout: single
title:  "Partial Derivative"
date:   2024-04-04 21:10:54 
categories: [ml, math]
author_profile: false
sidebar:
  nav: "ml"
---
## 1-8. 편미분

### 편미분

- 다변수 함수가 있을 때 하나의 변수에 대해서만 미분을 진행하는 것.

### 예시

$Let \enspace z = f(x,y) = xy^2$

$x$에 대한 편미분

$\dfrac{\partial f}{\partial x} = y^2$

$y$에 대한 편미분

$\dfrac{\partial f}{\partial y} = 2xy$

### 그라디언트

- 편미분을 벡터로 모아둔 것.
- 그라디언트는 항상 기울기가 가장 가파른 방향을 가르킨다.
- $\nabla f = \begin{bmatrix}\frac{\partial f}{\partial x} \\ \frac{\partial f}{\partial y}\end{bmatrix}$
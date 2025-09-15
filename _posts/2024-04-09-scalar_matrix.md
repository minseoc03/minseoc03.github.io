---
layout: single
title:  "Scalar derivative respect to Matrix"
date:   2024-04-09 21:03:54 
categories: [ml, math]
author_profile: false
sidebar:
  nav: "ml"
---
## 1-14. 스칼라를 행렬로 미분

### 입력이 행렬, 출력이 스칼라인 함수의 미분

$Let \enspace f(X) = tr(XA) \enspace$→ $tr()$은 행렬의 대각 원소들의 합이다.

$X = \begin{bmatrix}x_{11} & x_{12} \\ x_{21} & x_{22}\end{bmatrix}, \enspace dX = \begin{bmatrix}dx_{11} & dx_{12} \\ dx_{21} & dx_{22}\end{bmatrix}$

$df = \dfrac{\partial f}{\partial x_{11}}dx_{11}+\dfrac{\partial f}{\partial x_{12}}dx_{12}+\dfrac{\partial f}{\partial x_{13}}dx_{13}+\dfrac{\partial f}{\partial x_{14}}dx_{14}$

$= tr(\begin{bmatrix}dx_{11} & dx_{12} \\ dx_{21} & dx_{22}\end{bmatrix}\begin{bmatrix}\dfrac{\partial f}{\partial x_{11}} & \dfrac{\partial f}{\partial x_{12}} \\ \dfrac{\partial f}{\partial x_{21}} & \dfrac{\partial f}{\partial x_{22}}\end{bmatrix})$

$= tr(dX\dfrac{\partial f}{\partial X^T})$

$f'(x) = \enspace ?$

$\implies df = tr((x+dX)A) - tr(XA)$

$= tr(XA + dX\cdot A - XA)$

 $= tr(dx\cdot A) = tr(dX\dfrac{\partial f}{\partial X^T})$

$\therefore \dfrac{\partial f}{\partial X^T} = A$
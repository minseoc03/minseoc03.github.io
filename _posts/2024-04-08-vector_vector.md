---
layout: single
title:  "Vector derivative respect to vector"
date:   2024-04-08 21:01:54 
categories: [ml, math]
author_profile: false
sidebar:
  nav: "ml"
---
## 1-12. 벡터를 벡터로 미분

### 입력과 출력이 둘 다 벡터인 함수를 미분하는 경우

ex) $\vec f(\begin{bmatrix}x_1 & x_2\end{bmatrix}) = \begin{bmatrix}x_1x_2^2 & x_1 + x_2\end{bmatrix}$

### 스칼라를 벡터로 미분하는 과정을 2번 반복하면 된다.

$df_1 = \dfrac{\partial f_1}{\partial x_1}dx_1 + \dfrac{\partial f_1}{\partial x_2}dx_2$

$= \begin{bmatrix}dx_1 & dx_2\end{bmatrix}\begin{bmatrix}\frac{\partial f_1}{\partial x_1} \\ \frac{\partial f_1}{\partial x_2}\end{bmatrix}$

$df_2 = \dfrac{\partial f_2}{\partial x_1}dx_1 + \dfrac{\partial f_2}{\partial x_2}dx_2$

$= \begin{bmatrix}dx_1 & dx_2\end{bmatrix}\begin{bmatrix}\frac{\partial f_2}{\partial x_1} \\ \frac{\partial f_2}{\partial x_2}\end{bmatrix}$

$\implies d\vec f = \begin{bmatrix}df_1 & df_2\end{bmatrix} = \begin{bmatrix}dx_1 & dx_2\end{bmatrix}\begin{bmatrix}\frac{\partial f_1}{\partial x_1} & \frac{\partial f_2}{\partial x_1} \\ \frac{\partial f_1}{\partial x_2} & \frac{\partial f_2}{\partial x_2}\end{bmatrix} = d\vec x\dfrac{\partial \vec f}{\partial \vec x^T}$

→ 즉, 원하던 미분 값은 $\dfrac{\partial \vec f}{\partial \vec x^T}$이다.

### 예시

$Let \enspace \vec f(\vec x) = \vec xA \text{ where A is a certain matrix}$

$\implies d\vec f = (\vec x+ d\vec x)A - \vec xA$

$= d\vec x\cdot A$

$\therefore \dfrac{\partial \vec f}{\partial \vec x^T} = A$
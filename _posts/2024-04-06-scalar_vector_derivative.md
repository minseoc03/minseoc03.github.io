---
layout: single
title:  "Scalar derivative respect to vector"
date:   2024-04-06 21:03:54 
categories: [ml, math]
author_profile: false
sidebar:
  nav: "ml"
---
## 1-10. 스칼라를 벡터로 미분

### 미분 방법 도출

$Let \enspace f(x_1, x_2) = x_1x_2^2$

$df = \lim\limits_{\varDelta x_1, \varDelta x_2 \to 0} f(x_1+\varDelta x_1, x_2+\varDelta x_2) - f(x_1, x_2)$

 $= \lim\limits_{\varDelta x_1, \varDelta x_2 \to 0} f(x_1+\varDelta x_1, x_2+\varDelta x_2) - f(x_1, x_2) + f(x_1,x_2+\varDelta x_2) - f(x_1, x_2+\varDelta x_2)$

 $= \lim\limits_{\varDelta x_1, \varDelta x_2 \to 0} \dfrac{f(x_1+\varDelta x_1, x_2+\varDelta x_2)- f(x_1, x_2+\varDelta x_2)}{\varDelta x_1}\cdot\varDelta x_1 + \dfrac{f(x_1, x_2+\varDelta x_2)-f(x_1,x_2)}{\varDelta x_2}\cdot\varDelta x_2$

$= \dfrac{\partial f}{\partial x_1}dx_1 + \dfrac{\partial f}{\partial x_2}dx_2$

$= \begin{bmatrix}dx_1 & dx_2\end{bmatrix}\begin{bmatrix}\frac{\partial f}{\partial x_1} \\ \frac{\partial f}{\partial x_2}\end{bmatrix} = d\vec{x}\cdot\dfrac{\partial f}{\partial \vec{x}^T}$

- 우리가 원하는 미분 값은 $\dfrac{\partial f}{\partial \vec{x}^T}$. 즉, $df$를 구하고 $d\vec{x}$로 나누어주면 간단하게 스칼라를 벡터로 미분한 값을 얻을 수 있다.

### 위의 방법 활용

$Let \enspace f = \vec x \vec x^T$

$df = (\vec x+d\vec x)(\vec x + d\vec x)^T - \vec x \vec x^T$

 $= d\vec x \cdot \vec x^T + \vec x\cdot d\vec x^T + d\vec x\cdot d\vec x^T$ → $d\vec x \cdot d\vec x^T$는 매우작고 0으로 수렴하는 값이기에 무시해도 이상 없다.

$\implies d\vec x\cdot \vec x^T + \vec x \cdot d\vec x^T$

$= d\vec x\cdot \vec x^T + (\vec x\cdot d\vec x^T)^T$ → $\vec x \cdot d\vec x^T$는 스칼라 값이기에 전치를 취해주어도 변화가 없다.

$= d\vec x\cdot \vec x^T + d\vec x\cdot \vec x^T$

$= d\vec x\cdot 2\vec x^T$

$\therefore \dfrac{\partial f}{\partial \vec x^T} = 2\vec x^T$
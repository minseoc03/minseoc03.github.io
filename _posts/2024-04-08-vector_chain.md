---
layout: single
title:  "Vector derivative respect to vector (Chain Rule)"
date:   2024-04-08 21:03:54 
categories: [ml, math]
author_profile: false
sidebar:
  nav: "ml"
---
## 벡터를 벡터로 미분 (연쇄법칙)

### 예시

$Let \enspace \vec y = \vec xA, \enspace \vec z = \vec yB,$

$\text{What is } \dfrac{d\vec z}{d \vec x^T}?$

$\implies \vec z = \vec xAB$

$\implies d\vec f = (\vec x + d\vec x)AB - \vec xAB$

$= d\vec x AB$

$= d\vec x\dfrac{d\vec z}{d\vec x^T} \enspace \therefore \dfrac{d\vec z}{d\vec x^T} = AB$

### 예시(w/ 연쇄 법칙)

$d\vec y = d\vec x\dfrac{\partial \vec y}{\partial \vec x^T}$

$d\vec z = d\vec y\dfrac{\partial \vec z}{\partial \vec y^T}$

$\implies d\vec z = d\vec x\dfrac{\partial \vec y}{\partial \vec x^T}\dfrac{\partial \vec z}{\partial \vec y^T}$

- 기존 연쇄 법칙에서는 식이 구성된 순서에서 뒤로가며 미분을 구했다.
    - $(x^2+1)^2 \to x^2+1 \to x^2 \to x \\ \implies \frac{d(x^2+1)^2}{dx} = \frac{d(x^2+1)^2}{d(x^2+1)}\cdot\frac{d(x^2+1)}{dx^2}\cdot\frac{dx^2}{dx}$
- 하지만 여기서는 반대로 식이 구성되는 순서대로 미분을 구한다.
    - $\vec x \to \vec y \to \vec z$
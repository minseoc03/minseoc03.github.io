---
layout: single
title:  "Vector derivative respect to Matrix"
date:   2024-04-10 21:05:54 
categories: [ml, math]
author_profile: false
sidebar:
  nav: "ml"
---
## 1-16. 벡터를 행렬로 미분

### 예시

$Let \enspace \vec y = \vec xW \enspace$→ 간단한 DNN(Deep Neural Network)의 선형 구조

$\text{Then what is }f'(W)?$

$Let \enspace vec(W) = \vec w,$

$\begin{bmatrix}y_1 & y_2\end{bmatrix} = \begin{bmatrix}x_1 & x_2\end{bmatrix} \begin{bmatrix} w_{11} & w_{12} \\ w_{21} & w_{22} \end{bmatrix}$

$= \begin{bmatrix} w_{11}x_1+w_{21}x_1 & w_{12}x_1 + w_{22}x_2\end{bmatrix}$

$\dfrac{\partial \vec y}{\partial \vec w^T} = \begin{bmatrix} \dfrac{\partial y_1}{\partial w_{11}} & \dfrac{\partial y_2}{\partial w_{11}} \\ \dfrac{\partial y_1}{\partial w_{12}} & \dfrac{\partial y_2}{\partial w_{12}} \\ \dfrac{\partial y_1}{\partial w_{21}} & \dfrac{\partial y_2}{\partial w_{21}} \\ \dfrac{\partial y_1}{\partial w_{22}} & \dfrac{\partial y_2}{\partial w_{22}} \end{bmatrix} = \begin{bmatrix}x_1 & 0 \\ 0 & x_1 \\ x_2 & 0 \\ 0 & x_2\end{bmatrix} = \begin{bmatrix} x_1I_2 & x_2I_2 \end{bmatrix} = \vec x ⊗ I_2 \enspace$ → ⊗ is Kronecker prod.
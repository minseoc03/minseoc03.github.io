---
layout: single
title:  "Gradient Descent"
date:   2024-04-20 21:10:54 
categories: [ml, dl_theory]
author_profile: false
sidebar:
  nav: "ml"
---
## 3-3. 경사 하강법 (Gradient Descent)

### 경사 하강법의 개념

- $ax+b$의 식에서 $a,\space b$를 무작위로 먼저 정한다.
- 이후 $L$ 즉, 손실 값을 최소화 하는 방향으로 나아가자.

### 손실 값이 최소인 방향은?

- 그라디언트는 항상 가장 가파른 방향으로 나아간다는 것을 기억하자.
- 즉, 손실 함수에 대한 편미분을 구하고 그의 반대 방향으로 나아가면 된다.
    - $\nabla L = \begin{bmatrix}\dfrac{\partial L}{\partial a} \\ \dfrac{\partial L}{\partial b}\end{bmatrix}\bigg\rvert_{a = a_k, b = b_k}$→ 무작위로 정한 $a$와 $b$의 값이 $a_k$와 $b_k$이다.
    - $\begin{bmatrix}a_{k+1}\\b_{k+1}\end{bmatrix} = \begin{bmatrix}a_{k}\\b_{k}\end{bmatrix} - \nabla L$   → 이런 식으로 차차 나아가면서 최적의 값을 찾는 것.

### 학습률 (Learning Rate)

- 그라디언트에서 제시하는 크기가 생각보다 크기 때문에 최적점을 못 찾을 수도 있다.
- 그래서 그라디언트의 크기를 조절하기 위해 학습률을 추가한다.
    - $\begin{bmatrix}a_{k+1}\\b_{k+1}\end{bmatrix} = \begin{bmatrix}a_{k}\\b_{k}\end{bmatrix} - \alpha\nabla L$
- 학습률은 상수로 놔둬도 이상이 없다.
    - 그 이유는 최적점에 도달할수록 경사가 낮아지기에 그라디언트의 크기가 알아서 줄어들기 때문이다.
- 하지만 학습률을 변수로 놔두면 안된다는 건 아니다.
    - 학습률을 변수로 두는 것을 **스케줄링(Scheduling)**이라고 한다.
    - 점차 학습률을 줄어나가는 것인데 이에 대한 방법은 다양하다.

### 경사 하강법의 단점

- 너무 신중하게 방향을 정한다.
    - 모든 손실 값에 따라 방향을 정하기 때문에 속도가 너무 느리다.
- Local Minimum에 빠져서 그곳을 Global Maximum이라고 착각할수도 있다.
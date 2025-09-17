---
layout: single
title:  "Neural Network is MLE machine"
date:   2024-04-29 21:10:54 
categories: [ml, dl_theory]
author_profile: false
sidebar:
  nav: "ml"
---
## 5-4. 인공신경망은 MLE 기계다

### 인공신경망의 뿌리는 MLE

- 이전에 log-likelihood의 손실 함수를 보면…
    - $q^y(1-q)^{1-y}$ 라는 식이 나오는데 이는 사실 베르누이 분포를 따른다.
- Likelihood의 개념을 다시 되짚어보면…
    - $P(A\midB)$라는 확률 식에서 B를 변수로 두고 A가 나올 확률 값을 likelihood라고 하였다.
- 그렇다면…
    - $P(y\midq = f_w(\text{input})) = q^y(1-q)^{1-y}$ 라고 가정을 한다면 w의 함수로 보고 likelihood를 maximize하는 MLE의 관점으로 볼수가 있다.
    - 즉, log-likelihood는 NLL(negative log-likelihood)에 대한 MLE라는 것이다.
- 가정을 달리해보면 어떨까?
    - 베르누이 분포가 아니라 가우시안 분포라고 가정한 다음, 머신의 출력 $f_w(x_i)$를 평균 값 $\hat y$로 삼고 NLL 식을 세우자.
    - $\prod\limits_i(-\log \dfrac{1}{\sqrt{2\pi\sigma^2}}e^{-\dfrac{(y_i-\hat y_i)^2}{2\sigma^2}})$라는 식이 나오는데 이를 전개해보면
    - $\sum\limits_i\dfrac{(y_i-\hat y_i)^2}{2\sigma^2}$ 즉, 상수를 포함한 MSE가 튀어나오게 된다.
        - $\dfrac{1}{\sqrt{2\pi\sigma^2}}$는 상수라서 제외했다.
- 하지만 여기서는 사진 분류 문제이기에 출력 값이 이진형인 베르누이 분포인 $-\log q$를 사용한 것이다.
- 회귀 문제에서는 출력 값이 실수 형태로 다양하기에 MSE가 더 적당할 것이다.

<aside>
💡

- 결론은 가정을 어떻게 하냐의 차이지 결국은 인공신경망은 가정만 달리한 MLE를 한 것이다.
- 결국 학습이란 w에 대한 MLE를 한 것이다.
- $\hat w = \argmax\limits_wp(y_i\midf_w(x_i))$
</aside>
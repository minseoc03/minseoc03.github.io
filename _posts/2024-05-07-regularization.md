---
layout: single
title:  "Regularization"
date:   2024-05-07 21:10:54 
categories: [ml, dl_theory]
author_profile: false
sidebar:
  nav: "ml"
---

## 7-7. Regularization

### Regularization의 개념 설명

- $L = L + \lambda\mid\mid\vec w\mid\mid^p_p$
    - 만약 $p = 2$라면, L2 Regularization
    - 만약 $p = 1$라면, L1 Regularization
    - 여기서 $\lambda$는 우리가 정해야 하는 하이퍼파라미터다.
- Regularization은 가중치의 크기를 줄여가며 모델을 경량화 시켜 과적합을 방지해준다.
- L2의 경우 가중치의 크기에 제곱을 해주기 때문에 크기가 큰 가중치는 많이 줄여주고 크기가 작은 가중치는 변화가 거의 없다.
    - 즉, 모든 가중치가 거의 동일해지게 크기를 조정해준다.
- L1의 경우는 가중치의 크기에 상관없이 동일하게 줄여주기에 애초에 크기가 컸던 가중치만 남게 된다. 몇몇 노드는 connection이 없어지기도 한다.
    - 즉, 중요한 몇몇 가중치만 남긴다.

### Early Stopping

- 해당 방법도 과적합을 방지하기 위한 방법이다.
- Validation Loss가 가장 작을 때 학습을 멈추는 것이다.

![image 13.png](/assets/images/dl-theory/image%2013.png)

- 다음과 같이 Validation Loss가 가장 작을 때 train과 valid 사이의 에러 차이가 가장 적다.
- 그리고 가장 loss가 적은 에포크 이후로 더 훈련하면 valid loss가 오히려 커지면서 overfitting이 일어나게 된다.
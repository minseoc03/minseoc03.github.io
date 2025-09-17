---
layout: single
title:  "Linear Regression"
date:   2024-04-19 21:10:54 
categories: [ml, dl_theory]
author_profile: false
sidebar:
  nav: "ml"
---
## 3-2. 선형 회귀

### 선형 회귀 (Linear Regression)

- 입력과 출력 간의 관계(함수)를 (선형으로 가정하고) 추정하는 것
    - 처음 보는 입력에 대해서도 적절한 출력을 얻기 위함
- 예를 들어 키와 몸무게의 관계를 $ax+b$로 두고 $a, \enspace b$를 잘 추정하여 처음 보는 키로 몸무게를 추정하는 것
- 즉, 알아내야 하는 것은 최적의 weight(a), bias(b)이다.
- 하지만 최적이 무엇인가를 **판단하려면 기준이 필요**하다.
    - 그 기준이 **Loss Function**이다.
- 머신의 예측 $\hat y$와 실제 몸무게 $y$의 차이로 loss를 정의해보자.
    - 하지만 이러한 loss는 잘못된 방법이다.
    - 만약 양의 차이와 음의 차이가 같다면 loss가 0이 되어버린다.
- 즉, $\hat y$와 $y$의 차이를 제곱을 사용하여 부호를 동일화시켜준다.
    - 절댓값도 부호의 문제를 해결해줄 수 있지만 제곱의 경우가 차이가 커질수록 loss값을 기하급수적으로 올려주기에 오차에 더욱 민감하다.
    - 이러한 loss를 **MSE(Mean Squared Error)**라고 명한다.
- MSE를 최소화하는 a,b의 값을 어떻게 찾나?
    - 일일이 a,b값을 바꿔가며 loss function L의 그래프를 그리고 거기서의 최솟값을 찾는다.
    - 하지만 노드가 많아질수록 이는 비현실적인 방법.
    - 조금 더 스마트한 방법은 다음 페이지.
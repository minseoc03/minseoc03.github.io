---
layout: single
title:  "Weight Initialization"
date:   2024-04-20 21:12:54 
categories: [ml, dl_theory]
author_profile: false
sidebar:
  nav: "ml"
---
## 3-4. 가중치 초기화 (Weight Initialization)

### 선요약

- 가중치를 랜덤하게 0 근처로 초기화 해라.

### LeCun방법

- $w \sim U(-\sqrt{\dfrac{3}{N_{in}}}, \sqrt{\dfrac{3}{N_{in}}}) \enspace \text{or }w \sim N(0,\dfrac{1}{N_{in}})$
    - $N_{in} =$  입력 층의 노드 갯수
    - $N_{out} =$  출력 층의 노드 갯수
    - 균등 분포와 정규 분포의 평균과 분산은 똑같다

### Xavier (sigmoid/tanh 사용하는 신경망에 한해서)

- $w \sim U(-\sqrt{\dfrac{6}{N_{in}+N_{out}}}, \sqrt{\dfrac{6}{N_{in}+N_{out}}}) \text{ or }w \sim N(0, \dfrac{2}{N_{in}+N_{out}})$
    - 균등 분포와 정규 분포의 평균과 분산은 똑같다

### He (ReLU 사용하는 신경망에 한해서)

- $w \sim U(-\sqrt{\dfrac{6}{N_{in}}}, \sqrt{\dfrac{6}{N_{in}}}) \text{ or }N(0, \dfrac{2}{N_{in}})$
    - 균등 분포와 정규 분포의 평균과 분산은 똑같다

### 의문점

- 굳이 랜덤하게 초기화를 해야하나?
- 0이나 1로 초기화 하면 안되나?
    - 해당 의문에 대한 해결은 역전파(Back Propagation)을 이해하고 알아보자.
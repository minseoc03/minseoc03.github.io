---
layout: single
title:  "Attention"
date:   2024-05-16 21:10:54 
categories: [ml, dl_theory]
author_profile: false
sidebar:
  nav: "ml"
---

## 9-4. Attention

### RNN 문제의 해결 방안

- RNN의 고질적인 문제점인 기억력. 즉, 과거의 정보를 잊어버리는 문제는 Attention의 개념 도입으로 해결이 되었다.
- Attention이란 내적을 이용한 기법이다. 내적은 닮은 정도를 나타내는데 인코더의 $\vec h_t$와 디코더의 $\vec s_t$의 내적을 통해 닮은 정도의 정보를 가져가는 것이다.
- 하지만 이 또한 완벽한 해결책은 아니었다.

### RNN+Attention의 문제점

- Attention 또한 흐려지는 정보에 대해서는 취약하다.
    - 예를 들어서 ‘쓰다’라는 단어가 입력의 마지막에 온다고 가정하자.
    - 쓰다라는 단어는 주어에 따라 의미가 많이 달라진다. ‘돈을’, ‘모자를’, ‘맛이’, ‘글을’ 등등 이와 같이 주어에 따라 의미가 천차만별이다.
    - 하지만 첫번째 단어같은 경우 RNN의 특성상 잊혀지기 마련이고 이에 대해 Attention을 한다고 해도 성능이 좋진 않을 것이다.
- 또한 한 방향으로만 흘러가는 RNN의 특성상 뒷 단어를 보고 단어를 유추해야하는 영어 같은 경우 문제가 된다.
    - 예를 들어 an instructor인데 an을 유추하려면 뒷 단어인 instructor를 봐야한다.
    - 이에 대한 문제는 bidirectional RNN을 사용을 해야한다.

### Transformer의 등장

- Self-Attention이라는 개념이 도입된 후 RNN의 구조를 완전히 버리고 Transformer라는 구조가 자리잡게 되었다.

![image.png](/assets/images/dl-theory/image.png)

- Self Attention이란?
    - 서로가 서로의 Attention을 계산 한후 해당 정보를 이용하는 것이다.
    - 다음과 같이 입력 데이터끼리 서로 Attention을 계산한다.
    - 같은 단어끼리의 내적은 값이 가장 높을테고 ‘저는’뒤에는 ‘강사’라는 단어가 ‘입니다’ 보다 더 어울리니까 이에 대한 내적 값이 더 높을 것이다.
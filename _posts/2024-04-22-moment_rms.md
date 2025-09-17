---
layout: single
title:  "Momentum vs. RMSProp"
date:   2024-04-22 20:10:54 
categories: [ml, dl_theory]
author_profile: false
sidebar:
  nav: "ml"
---
## Momentum vs. RMSProp

### Momentum

![image 28.png](/assets/images/dl-theory/image%2028.png)

- SGD의 개념은 유지하되 관성이라는 개념을 추가했다.
- 즉, 과거의 그라디언트의 방향을 기억해서 관성을 적용한다.
- 위 그림을 보다싶이 좌우로 가려는 힘은 상쇄가 되고 앞으로 가는 힘만 누적이 되어 최적점을 찾아간다.

### RMSProp

- 예를 들어 $L$이라는 손실 함수에 변수가 $a$와 $b$가 있다고 하자.
- 그러면 RMSProp은 각각의 변수에 학습률을 따로 적용한 개념이라고 보면 된다.
- 그라디언트의 크기가 큰 변수는 작은 학습률을 적용하고 그라디언트가 작은 변수는 큰 학습률을 적용 시켜준다.
- 조금 더 추상적으로 설명해보자면 그라디언트의 크기가 크다는 것은 해당 변수 쪽은 매우 가파르다는 것이고 그라이디언트가 작은 변수는 경사가 완만하다는 것이다.
- 즉, 완만한 곳에서 더 돌아보고 경사가 가파른 곳은 조심하겠다는 의미이다.

![image.png](/assets/images/dl-theory/image%201%2018.png)
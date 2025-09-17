---
layout: single
title:  "CNN Feature Map"
date:   2024-05-11 21:10:54 
categories: [ml, dl_theory]
author_profile: false
sidebar:
  nav: "ml"
---

## 8-5. CNN의 Feature Map 분석

### Feature Map의 진화

![image 8.png](/assets/images/dl-theory/image%208.png)

- CNN의 기본적인 구조는 Convolution과 Pooling을 반복을 한다.
- Convolution을 반복할 때에는 갈 수록 필터의 갯수를 늘려가며 더 많은 특징을 알아볼수 있도록 한다.
- Pooling도 반복을 해주며 점점 범위를 넓혀준다.

![image.png](/assets/images/dl-theory/image%201%206.png)

- 최종적으로 나온 피처 맵들을 겹쳐서 보면 특정 특징이 있는 곳만 색깔이 뚜렷하게 나와 특징을 구별해낼 수 있을 것이다.

- 이후 MLP를 통과시켜 최종적으로 해당 이미지가 어떠한 동물인지를 예측한다.
    - 즉, CNN이라고 MLP가 없는 것이 아니라 마지막 출력층에는 FC가 필요하다.

### Feature Map 실제 모습

![image.png](/assets/images/dl-theory/image%202%203.png)

- CIFAR-10 데이터 셋을 이용하여 간단하게 이미지 분류 CNN 모델을 만들어보자.
- 첫번째 Conv. Block 에서 nn.Conv2d(3,32,3,padding =1) 라는 명령어를 입력했다.
    - 이는 in-channel이 3개, out-channel이 32개, 커널의 사이즈가 3x3이라는 것이다.
    - 즉, 32개의 특징을 뽑아보겠다는 말이다.
- 이후 두번째 블록에서는 32개에서 64개로, 세번째 블록에서는 64개에서 128개로 필터의 갯수를 늘렸다.
- 각 합성곱 연산마다 BN과 ReLU를 써주며 vanishing gradient문제를 해결했다.

![image.png](/assets/images/dl-theory/image%203%202.png)

- 첫번째 합성곱 블록을 통과한 뒤의 사진이다.
- 입력 이미지는 강아지이며 강아지의 외곽 선을 표시한 걸 알아볼 수 있다.

![image.png](/assets/images/dl-theory/image%204%201.png)

- 두번째 합성곱 블록을 통과한 뒤의 출력물이다.

- 첫번째와 달리 조금 더 디테일하게 강아지의 모습을 뽑아낸 것을 볼수 있다.

![image.png](/assets/images/dl-theory/image%205%201.png)

- 128개의 피처맵 중 일부만 가져왔다.

- 세번째 합성곱을 통과하니 특정 부분에만 값이 높게 나와 흰색으로 표시가 되는데 이미지의 특징의 위치를 표시하는 것 같다.

- 풀링을 계속함에 따라 강아지의 모습은 사라지고 예를 들어 눈이라면 눈의 위치가 흰색으로 표시되는 것을 볼수 있다.

![image.png](/assets/images/dl-theory/image%206%201.png)

- 마지막에 나오는 128개의 피처 맵을 모두 겹치니 강아지의 특징을 잘 뽑아낸 것을 볼수가 있다.
- 배경은 값을 줄이고 강아지의 특징이 있는 곳만 값이 높아 노란색으로 표시가 되는 것을 볼수가 있다.

![image.png](/assets/images/dl-theory/image%207%201.png)
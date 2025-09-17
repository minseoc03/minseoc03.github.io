---
layout: single
title:  "Padding, Pooling, Striding"
date:   2024-05-10 21:10:54 
categories: [ml, dl_theory]
author_profile: false
sidebar:
  nav: "ml"
---

## 8-4. Padding, Pooling, Striding

### Padding

- 합성곱 연산을 하다보면 커널의 사이즈가 1x1이 아닌 이상 입력 데이터에 비해 출력 데이터의 사이즈는 줄어든다.
- 이러한 연산을 반복하면 결국 픽셀 1개만이 남아버릴 것이다.

![image 9.png](/assets/images/dl-theory/image%209.png)

- 그래서 입력 데이터에 0 (다른 값도 됨)을 주위에 채워넣어 출력 값의 사이즈가 유지 되도록 하는 기법을 Padding 기법이라고 한다.

### Stride

- Stride는 커널이 이미지를 스캔하면서 몇 칸씩 움직일까를 나타낸다.
- 우리가 여태껏 봤던 예시들은 Stride = (1,1)인 상황이다.
    - 열 (즉, 오른쪽으로) 한칸 씩, 행 (즉, 아래로) 한줄 씩

![image.png](/assets/images/dl-theory/image%201%207.png)

- 다음과 같이 (2,2)로 설정 시, 오른쪽으로 2칸 씩, 아래로 2칸 씩 움직이며 합성곱 연산을 실시한다.
- Stride가 커질 수록 출력 피처 맵 사이즈도 줄어든다.

### Pooling

![image.png](/assets/images/dl-theory/image%202%204.png)

- Max Pooling
    - 위의 예시대로 만약 사이즈가 (2,2)고, stride가 (2,2)면 2x2사이즈로 돌면서 각 커널의 최댓값을 꺼내오는 방법이다.
- Average Pooling
    - 위에 서술한 MaxPooling과 같은 방식이지만 최댓값이 아니라 해당 커널에 있는 값의 평균을 뽑아오는 것이다.
- 이러한 풀링 기법을 사용할 시 적은 값으로 넓은 범위를 대표할 수 있게 한다.
- 그리고 풀링 각 채널 별로 개별로 진행되기 때문에 채널 갯수는 유지된다.
- GAP (Global Average Pooling)
    - 만약 사이즈가 입력 이미지 사이즈와 같다면 그냥 입력 이미지 전체의 평균 값 1개 만을 출력하는 것이다.
    - RGB 이미지에서 GAP을 시행한다면 출력 값은 3x1x1일 것이다.
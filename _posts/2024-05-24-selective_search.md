---
layout: single
title:  "Selective Search"
date:   2024-05-24 21:10:54 
categories: [ml, object_detection]
author_profile: false
sidebar:
  nav: "ml"
---
## Selective Search

### Region Proposal의 예시

Region Proposal은 지난 노트에서도 언급했지만 Sliding Window의 개선책으로 미리 객체가 있을 후보영역을 정해두는 방법을 말한다.

Selective Search란 Region Proposal 방법 중 초기 object detection 모델인 R-CNN에서 사용된 방법이다.

### Selective Search 과정

1. 초기 이미지에서 Over-Segmentation을 통해 객체가 있을만한 모든 구역을 세세하게 나타낸다.
2. 이후 알고리즘을 통해 유사도가 높은 segmentation들을 하나의 segmentation으로 합쳐준다.
3. 2번의 과정을 반복을 해주며 최종 후보 영역을 도출해낸다.

2번 과정에 있는 알고리즘은 다음과 같다.

![image 5.png](/assets/images/object_detection/image%205.png)

- $R = \{r_1, \cdots , r_n\}$은 최초 n개로 나누어진 영역 $r_n$의 집합을 말한다.
- $S$는 영역들 간의 유사도 집합을 말한다.
- 유사도가 가장 높은 $r_i$와 $r_j$를 합친 $r_t$ 영역을 새로 생성한다.
- $S$에서 $r_i$와 $r_j$가 포함된 유사도를 제거한다.
- $r_t$를 이용하여 새로운 유사도를 구하여 새로운 유사도 집합 $S_t$를 생성한다.
- $R$에 $r_t$를 추가하고 $S$에 $S_t$를 추가한다.

![image.png](/assets/images/object_detection/image%201%201.png)

다음과 같이 2번 과정을 반복할수록 유사도가 높은 영역들이 합쳐지며 영역의 갯수가 줄어드는 것을 볼 수 있다.
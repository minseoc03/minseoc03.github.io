---
layout: single
title:  "Sliding Window"
date:   2024-05-22 21:10:54 
categories: [ml, object_detection]
author_profile: false
sidebar:
  nav: "ml"
---

## Sliding Window 기법

### Object Detection 구현의 문제점

저번 노트에서 언급했듯이 기존 CNN 구조를 가지고서는 2개 이상의 객체를 식별해내고 구분해내야되는 object detection을 하기에는 무리가 있다.

이에 대한 문제를 해결하기 위해 다음과 같은 의견이 제시되었다. **“Feature Map이 복잡해는 것이면 아예 이미지를 작은 구역으로 나누어 보면 되지 않겠는가?”**

### Sliding Window

Sliding Window는 위에서 제시된 의견을 따라 CNN에서 커널이 이미지를 훑어보듯 좌측 상단에서 부터 우측 하단까지 이미지를 잘라본다.

![image 4.png](/assets/images/object_detection/image%204.png)

하지만 해당 방법 같은 경우 또다른 문제점이 발생한다.

만약 **window 사이즈 보다 이미지 사이즈가 작거나** **원하는 객체가 window 사이즈 안에 다 안들어가면** 어떻게 할것인가?

이에 대한 문제를 해결하기 위해 이미지의 크기를 달리해서 똑같은 작업을 반복하였다.

### Sliding Window의 문제점

위와 같이 일정한 사이즈의 window로 이미지를 자른 다음에 해당 작업을 이미지 사이즈를 달리해서 반복하는 작업은 듣기만 해도 **연산량이 엄청난 비효율적인 작업**임을 알 수 있다.

그래서 새로운 방안이 나온 것이 **Region Proposal** 이라는 방법이다.
해당 방법은 객체가 있을만한 구역을 미리 찾아주는 작업을 말한다. 해당 방법에 대해서는 다음 노트에 적어보도록하겠다.
---
layout: single
title:  "Tokenizing"
date:   2024-09-12 21:10:54 
categories: [ml, transformer]
author_profile: false
sidebar:
  nav: "ml"
---

## 토크나이징 간단 정리

### 토큰의 개념

- 토큰이란 시계열 데이터의 하위 구조라고 생각하면 된다.
- 즉, 토큰이 모여 하나의 데이터를 만드는 것이다.
- RNN에서는 한 개의 토큰이 하나의 입력 값인 것이다.

### 토크나이징

- 데이터를 어떠한 기준으로 토큰을 나누는 것을 토크나이징이라고 한다.
- 예를 들어 Hello 라는 단어를 토크나이징을 하면…
    - H, e, l, l, o 의 형식으로 5개의 토큰이 생겨난다.
- 요즘은 대부분 sub-word tokenization을 사용한다.

### Sub-word Tokenization

- 예를 들어 Pre-trained라는 단어를 토크나이징을 해보면 다음과 같이 된다.
    - Pre, train, -ed
- 해당 방법을 사용하는 이유는 다음과 같다.
    - 예를 들어 pretrained라는 단어가 테스트 데이터에 처음으로 나오고 단어 단위로 토크나이징을 진행했다면 다음 단어의 예측이 상당히 어려울 것이다.
    - 하지만 pre, train, -ed 로 나누어 본다면 학습 데이터에 있는 pre, train, -ed의 뜻을 각각 학습하여 pre-trained의 뜻을 유추하기 훨씬 편할 것이다.
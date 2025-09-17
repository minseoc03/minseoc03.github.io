---
layout: single
title:  "Chain Rule"
date:   2024-04-04 21:01:54 
categories: [ml, math]
author_profile: false
sidebar:
  nav: "ml"
---
## 1-7. 연쇄 법칙

### 연쇄법칙

- 미분을 연쇄하여 연산하는 방법으로 여러 개의 미분을 한번에 진행하도록 도와주는 법칙이다.

### 예시 (연쇄법칙 미적용)

- $y = (x^2+1)^2, \enspace f'(x) =\enspace ?$
- $f'(x) = \lim\limits_{\varDelta x \to 0}\dfrac{((x+\varDelta x)^2+1)^2 - (x^2+1)^2}{\varDelta x}$
- 해당 식은 너무 복잡하여 푸는데 시간이 너무 오래 걸림.

### 예시 (연쇄법칙 적용)

- $(x^2+1)^2 \to x^2+1 \to x^2 \to x$
- $\dfrac{d(x^2+1)^2}{dx} = \dfrac{d(x^2+1)^2}{d(x^2+1)} \dfrac{d(x^2+1)}{dx^2} \dfrac{dx^2}{dx} \\ =2(x^2+1)\cdot 1 \cdot 2x \\ =2(x^2+1)2x$

### 연쇄법칙 증명

- $Let \enspace y = f(x) \enspace \&  \enspace z = g(y) = g(f(x))$
- $\dfrac{dz}{dx} = \lim\limits_{\varDelta x \to 0}\dfrac{g(f(x+\varDelta x)) -g(f(x))}{\varDelta x} \\ =\lim\limits_{\varDelta x \to 0}\dfrac{g(f(x+\varDelta x)) -g(f(x))}{f(x+\varDelta x) - f(x)} \cdot \dfrac{f(x+\varDelta x)-f(x)}{\varDelta x} \\ =\dfrac{dz}{dy}\dfrac{dy}{dx}$
---
layout: single
title:  "Derivative"
date:   2024-04-04 21:00:54 
categories: [ml, math]
author_profile: false
sidebar:
  nav: "ml"
---

## 1-6. 미분 (도함수)

### 미분

- 미분 = 순간 기울기
- 기울기 = $\frac{\varDelta y}{\varDelta x}$
- 1차 함수의 경우 어느 부분이든 기울기가 동일하다
- 2차 함수는 $\varDelta x$가 작아질수록 기울기가 달리진다

### 그렇다면 순간 기울기는 어떻게 구할까?

- 순간 기울기의 정의: $\lim\limits_{\varDelta x \to 0} \frac{f(x+\varDelta x) - f(x)}{\varDelta{x}}$
- 예시로 $x$가 1일 때의 $y = x^2$의 순간 기울기는?

$\lim\limits_{\varDelta x \to 0} \frac{f(1+\varDelta x) - f(1)}{\varDelta{x}} = \lim\limits_{\varDelta x \to 0} \frac{(1+\varDelta x)^2 - 1^2}{\varDelta{x}} = \lim\limits_{\varDelta x \to 0} \frac{\varDelta x^2 +2\varDelta{x}}{\varDelta{x}} = \lim\limits_{\varDelta x \to 0} \varDelta{x}+2  = 2$

- $x$가 변수 그대로 남아있을때의 순간 기울기는?

$\lim\limits_{\varDelta x \to 0} \frac{(x+\varDelta x)^2 - x^2}{\varDelta{x}} = \lim\limits_{\varDelta x \to 0} \frac{x^2 + 2x\varDelta{x}+\varDelta{x}^2-x^2}{\varDelta{x}} = \lim\limits_{\varDelta x \to 0} 2x+\varDelta{x} = 2x = f'(x)$

⇒ 여기서 구한 값이 바로 도함수.

### 도함수 공식

- $x^n \implies nx^{n-1}$
- $log_2x\implies \frac1{ln2}\frac1x$

- $e^x\implies e^x$
- $af(x) \implies af'(x)$

- $ln\,x\implies \frac1x$
- $f(x) + g(x)\implies f'(x)+g'(x)$
- $f(x)g(x) \implies f'(x)g(x) + f(x)g'(x)$
---
layout: single
title:  "MacClaurin & Taylor Expansion"
date:   2024-04-05 21:03:54 
categories: [ml, math]
author_profile: false
sidebar:
  nav: "ml"
---
## 1-9. 매클로린 & 테일러 급수

### 매클로린(MacClaurin) 급수

- 어떤 임의의 함수를 다항 함수로 나타내는 것.
- ex) $\cos x \implies x+x^2+x^3+x^4+\cdots$

### 왜 매클로린 급수를 사용할까?

- 다항 함수로 만듦으로써 전 구간을 미분이 가능하게 만든다.

[https://64.media.tumblr.com/e5580aeaf00adb1673a59cf63391370b/tumblr_n9ejwiGSbi1tzs5dao1_r2_1280.gifv](https://64.media.tumblr.com/e5580aeaf00adb1673a59cf63391370b/tumblr_n9ejwiGSbi1tzs5dao1_r2_1280.gifv)

- 문제는 각 항의 계수가 얼마여야 완벽히 $\cos x$를 흉내낼 수 있는가?
- $\cos x \implies C_0+ C_1x+C_2x^2+C_3x^3+C_4x^4+\cdots$

### 매클로린 급수 계수 구하기

- $C_0 = \enspace ?$
    - 양변의 $x$에 0을 대입
    $\cos 0 = C_0 \implies C_0 = 1$
- $C_1 = \enspace ?$
    - 미분하기
    $-\sin x = C_1+2C_2x+3C_3x^2+4C_4x^3+\cdots$
    - $x$에 0 대입
    $-\sin 0 = C_1 \implies C_1 = 0$
- $C_2 = \enspace ?$
    - 미분하기
    $-\cos x = 2C_2+6C_3x+12C_4x^2+\cdots$
    - $x$에 0 대입
    $-\cos 0 = 2C_2 \implies 2C_2 = -1 \implies C_2 = -\frac12$

### 예시로 구한 조합 말고 다른 조합은 없는가?

- 없다. 각 항이 독립적이라 다른 조합은 존재하지 않는다.

### 매클로린 급수 일반식

$C_n = \dfrac{f^n(0)}{n!}$

### 테일러 급수

- 매클로린 급수와 달리 0에 중심을 맞추는게 아니라 a라는 임의의 점에 중심을 맞춘다.
$f(x) \implies C_0+ C_1(x-a)+C_2(x-a)^2+C_3(x-a)^3+C_4(x-a)^4+\cdots$

### 테일러 급수 일반식

$C_n=\dfrac{f^n(a)}{n!}$

### 예시

$y = e^x$

$\begin{cases}f'(x) = e^x \\ f''(x) = e^x \\ f'''(x)=e^x\\ \vdots \end{cases}$

$\begin{cases}f'(0) = 1 \\ f''(0) = 1 \ f'''(0)=1 \\ \vdots\end{cases}$

$\therefore y=e^x \\ \implies 1+x+ \frac{1}{2!}x^2+ \frac{1}{3!}x^3+\frac{1}{4!}x^4+ \cdots$

$y = \ln x\enspace(a = 1)$

$\begin{cases}f(x) = \ln x \\ f'(x) = \frac1x \\ f''(x)=-\frac{1}{x^2} \\ f'''(x)= \frac{2}{x^3} \\ f''''(x)=-6x^{-4} \\ \vdots \end{cases}$

$\begin{cases}f(a) = 0 \\ f'(a) = 1 \\ f''(a)=-1 \\ f'''(a)=2 \\ f''''(a)=-6 \\ \vdots\end{cases}$

$\therefore y=\ln x \\ \implies (x-a)- \frac12(x-a)^2+ \frac13(x-a)^3 \cdots$

- 하지만 $\ln x$는 $x\geq2$부터 수렴하지 않는다.
- 즉, 테일러 급수는 무적이 아니다.

### 테일러 급수 수렴 구간

$\lim\limits_{n\to\infty}\mid\dfrac{p_{n+1}}{p_n}\mid < 1$

### 수렴 구간 예시

$Let \enspace y=\ln x$

$\implies \lim\limits_{n\to\infty}\mid\dfrac{\frac{1}{n+1}(x-1)^{n+1}}{\frac{1}{n}(x-1)^n}\mid<1$

$\implies \lim\limits_{n\to\infty}\mid\dfrac{n}{n+1}(x-1)\mid<1$

$\implies -1 < x-1 <1$

$\implies 0<x<2$
---
layout: single
title:  "Gradients"
date:   2024-04-07 21:03:54 
categories: [ml, math]
author_profile: false
sidebar:
  nav: "ml"
---
## 1-11. 그라디언트

### 왜 그라디언트는 가장 가파른 방향을 향할까?

$L(w_1, w_2) \enspace where \enspace w = w_k$ 

- 테일러 급수로 변환

$L(\vec w) = C_0 + C_1(w_1 - w_{1k})+C_2(w_2 - w_{2k})$

 $= C_0+\begin{bmatrix}w_1 - w_{1k} & w_2-w_{2k}\end{bmatrix}\begin{bmatrix}C_1 \\ C_2\end{bmatrix}$

$\implies C_n = \dfrac{f^n(a)}{n!}\text{, so }\begin{bmatrix}C_1 \\ C_2\end{bmatrix} = \begin{bmatrix} \dfrac{\partial L}{\partial w_1} \bigg\rvert_{w_1 = w_{1k}}\\ \dfrac{\partial L}{\partial w_2} \bigg\rvert_{w_2 = w_{2k}}\end{bmatrix}$ 

$\implies L(\vec w) \cong L(\vec {w_k}) + (\vec w - \vec {w_k})\dfrac{\partial L}{\partial \vec w^T}\bigg\rvert_{\vec w = \vec {w_k}} \text{, where }\vec {w}^T = \begin{bmatrix}w_1 \\ w_2\end{bmatrix} \& \enspace \vec {w_k} = \begin{bmatrix} w_{1k} & w_{2k} \end{bmatrix}$

→ 근사값으로 표시한 이유는 테일러 급수를 1차까지만 전개했기 때문이다.

- $\vec{w}_{k+1} = \vec{w}_k + \vec\varDelta$ 일때의 함수 구하기

$L(\vec w_{k+1}) \cong L(\vec w_k) + (\vec w_k + \vec\varDelta - \vec w_k)\dfrac{\partial L}{\partial \vec w^T}\bigg\rvert_{\vec w = \vec {w_k}} = L(\vec w_k) +  \vec\varDelta\dfrac{\partial L}{\partial \vec w^T}\bigg\rvert_{\vec w = \vec {w_k}}$

$\implies L(\vec w_{k+1}) - L(\vec w_k) = \vec\varDelta \dfrac{\partial L}{\partial \vec w^T}\bigg\rvert_{\vec w = \vec {w_k}}$

$Let \enspace\dfrac{\partial L}{\partial \vec w^T}\bigg\rvert_{\vec w = \vec {w_k}} = \vec g$

$\implies L(\vec w_{k+1}) - L(\vec w_k) = \vec\varDelta \bullet \vec g$           ($\vec\varDelta$는 행벡터, $\vec g$는 열벡터이므로 이는 내적이다.)

→ 두 함수의 값의 차이가 클수록 경사가 가파르다는 뜻이다.

→ 이 차를 최대화할려면 $\vec\varDelta$와 $\vec g$의 내적이 최대화 되어야 한다

→ $\vec g$는 미분 값이므로 고정 값이고 $\vec \varDelta$는 변화 가능한 변수이다.

→ $\vec\varDelta \bullet \vec g= ||\vec\varDelta||||\vec g||\cos\theta$ 이므로 $\theta = 0$일 때 내적이 최대가 되고 경사가 가장 가파르다는 것을 알 수 있다.

→ 즉, $\vec g$(그라디언트)와 $\vec\varDelta$(변화량)이 방향이 같아야 경사가 가장 가파르다.

→ 결론은 $\vec\varDelta$만큼 update할 때 그라디언트 방향으로 update하는게 L(손실 함수)를 가장 최대화 할 수 있는 방법이다.
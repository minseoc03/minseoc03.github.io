---
layout: single
title:  "Back Propagation with Matrix Derivatives"
date:   2024-04-26 21:10:54 
categories: [ml, dl_theory]
author_profile: false
sidebar:
  nav: "ml"
---
## 4-3. 행렬 미분을 이용한 역전파

### 왜 행렬 미분을 사용하나?

- 이전 4-2강에서 역전파를 배울 때는 모든 루트를 고려해서 다 더해주어야 정확한 편미분 값이 나온다라고 배웠다.
- 하지만 벡터로 식을 표현하고 행렬로 미분한다면 위와 같은 상황을 고려하지 않아도 된다.

### 행렬 미분을 이용한 역전파

- 4-2강과 동일한 신경망을 예시로 들며 이해해보자.
- 입력 층 노드에서 가중치 $W_1$이 곱해져서 나온 벡터를 $\vec d_1$
은닉 층 활성화 함수를 통과한 값이 $\vec n_1$
은닉 층 노드에서 가중치 $W_2$가 곱해져서 나온 벡터를 $\vec d_2$
출력 층 활성화 함수를 통과한 값이 $\vec n_2$ 라고 가정하자.
- 수식으로 풀어쓰면…
    - $\vec d_1 = \vec n_0W_1+\vec b_1$
    - $\vec n_1 = \vec f_1(\vec d_1)$
    - $\vec d_2 = \vec n_1W_2+\vec b_2$
    - $\vec n_2 = \vec f_2(\vec d_2)$
- 또한 손실 함수 (MSE)도 벡터로 표현이 가능하다.
    - $L = (\hat y_1 - y_1)^2 + (\hat y_2 - y_2)^2$
    $= (\vec n_2-\vec y)(\vec n_2-\vec y)^T$
    $= (\begin{bmatrix}\hat y_1 & \hat y_2\end{bmatrix} - \begin{bmatrix}y_1 & y_2\end{bmatrix})(\begin{bmatrix}\hat y_1 \\ \hat y_2\end{bmatrix} - \begin{bmatrix}y_1 \\ y_2\end{bmatrix})$
    $= \begin{bmatrix}\hat y_1 - y_1 & \hat y_2 - y_2\end{bmatrix}\begin{bmatrix}\hat y_1 - y_1 \\ \hat y_2 - y_2\end{bmatrix}$
    $= (\hat y_1 - y_1)^2 + (\hat y_2 - y_2)^2$
- 먼저 행렬로 미분을 진행 할 때에는 행렬을 벡터화 해줘야한다.
    - $\vec w_1 = vec(W_1), \vec w_2 = vec(W_2)$
- 벡터를 행렬로 미분할때 연쇄법칙이 사용되면 구성되는 순서 그대로 연쇄법칙이 적용된다.
    - $\dfrac{\partial L}{\partial \vec w_2^T} = \dfrac{\partial \vec d_2}{\partial \vec w_2^T}\dfrac{\partial \vec n_2}{\partial \vec d_2^T}\dfrac{\partial L}{\partial \vec n_2^T}$
- 위에서 만들어진 편미분 값을 풀어써보자.
    - $dL = d\vec n_2(\vec n_2 - \vec y)^T + (\vec n_2 - \vec y)d\vec n_2^T$
    $= d\vec n_2(\vec n_2 - \vec y)^T + ((\vec n_2 - \vec y)d\vec n_2^T)^T$
    $= 2d\vec n_2(\vec n_2 - \vec y)^T$
    $= d\vec n_2\space2(\vec n_2 - \vec y)^T$
    $\therefore \dfrac{\partial L}{\partial \vec n_2^T} = 2(\vec n_2-\vec y)^T$
    - $\dfrac{\partial \vec n_2}{\partial \vec d_2^T} = \begin{bmatrix}\dfrac{\partial n_{21}}{\partial d_{21}} & \dfrac{\partial n_{22}}{\partial d_{21}} \\ \dfrac{\partial n_{21}}{\partial d_{22}} & \dfrac{\partial n_{22}}{\partial d_{22}}\end{bmatrix} = \begin{bmatrix}\dfrac{\partial n_{21}}{\partial d_{21}} & 0 \\ 0 & \dfrac{\partial n_{22}}{\partial d_{22}}\end{bmatrix} = diag(\vec f'_2(\vec d_2))$
    - $\dfrac{\partial \vec d_2}{\partial \vec w_2^T} = \vec n_1^T \otimes I$  ⇒ 벡터를 행렬로 미분을 하면 크로네커 곱이 나온다고 배웠음.
    - $\therefore \dfrac{\partial L}{\partial \vec w_2^T} = (\vec n_1^T \otimes I) (diag(\vec f'_2(\vec d_2)))(2(\vec n_2-\vec y)^T)$
- 한 층 더 깊게 가서 편미분 값을 구해보자
    - $\dfrac{\partial L}{\partial \vec w_1^T} = \dfrac{\partial \vec d_1}{\partial \vec w_1^T}\dfrac{\partial \vec n_1}{\partial \vec d_1^T}\dfrac{\partial \vec d_2}{\partial \vec n_1^T}\dfrac{\partial \vec n_2}{\partial \vec d_2^T}\dfrac{\partial L}{\partial \vec n_2^T}$
    - $\dfrac{\partial L}{\partial \vec n_2^T} = 2(\vec n_2-\vec y)^T$
    - $\dfrac{\partial \vec n_2}{\partial \vec d_2^T} = diag(\vec f'_2(\vec d_2))$
    - $d\vec d_2 = d\vec n_1W_2$
    $\therefore \dfrac{\partial \vec d_2}{\partial \vec n_1} = W_2$
    - $\dfrac{\partial \vec n_1}{\partial \vec d_1^T} = diag(\vec f'_1(\vec d_1))$
    - $\dfrac{\partial \vec d_1}{\partial \vec w_1^T} = \vec n_0^T \otimes I$
    - $\therefore \dfrac{\partial L}{\partial \vec w_1^T} = (\vec n_0^T \otimes I)(diag(\vec f'_1(\vec d_1)))(W_2)(diag(\vec f'_2(\vec d_2)))(2(\vec n_2-\vec y)^T)$
    
    <aside>
    💡
    
    행렬 미분을 사용해도 ‘액웨액웨액웨…액앤’ 이라는 패턴이 똑같이 나온다.
    
    </aside>
    

### 스칼라를 행렬로 미분해서 역전파 구현

- 사실 위에서 진행한 방법은 편법에 가깝다.
    - 행렬을 vectorize해서 미분을 진행했기 때문에
- $\dfrac{\partial L}{\partial \vec w_1} = \dfrac{\partial \vec d_1}{\partial \vec w_1^T}\dfrac{\partial \vec n_1}{\partial \vec d_1^T}\dfrac{\partial \vec d_2}{\partial \vec n_1^T}\dfrac{\partial \vec n_2}{\partial \vec d_2^T}\dfrac{\partial L}{\partial \vec n_2^T}$에서 $\dfrac{\partial \vec n_1}{\partial \vec d_1^T}\dfrac{\partial \vec d_2}{\partial \vec n_1^T}\dfrac{\partial \vec n_2}{\partial \vec d_2^T}\dfrac{\partial L}{\partial \vec n_2^T}$를 $\vec v^T$로 치환해보자
- $\dfrac{\partial L}{\partial \vec w_1^T} = (\vec n_0^T \otimes I)\vec v^T$ 라는 식이 된다.
- 이를 풀어써보자면…
    - $\vec n_0^T = \begin{bmatrix}x_1 \\ x_2\end{bmatrix}, \vec v^T = \begin{bmatrix}v_1 \\ v_2 \\ v_3\end{bmatrix}$ 라고 하자.
    - $(\vec n_0^T \otimes I)\vec v^T = \begin{bmatrix}x_1 & 0 & 0 \\ 0 & x_1 & 0 \\ 0 & 0 & x_1 \\ x_2 & 0 & 0 \\ 0 & x_2 & 0 \\ 0 & 0 & x_2\end{bmatrix} \begin{bmatrix}v_1 \\ v_2 \\ v_3 \end{bmatrix} = \begin{bmatrix}x_1v_1 \\ x_1v_2 \\ x_1v_3 \\ x_2v_1 \\ x_2v_2 \\ x_2v_3\end{bmatrix}$
- 여기서 $W_1$ 가중치 행렬에 크기 대로 다시 나열하면… (크기는 2x3)
    - $\begin{bmatrix}x_1v_1 & x_1v_2 & x_1v_3 \\ x_2v_1 & x_2v_2 & x_2v_3\end{bmatrix} = \begin{bmatrix}x_1 \\ x_2\end{bmatrix} \begin{bmatrix}v_1 & v_2 & v_3\end{bmatrix} = \vec n_0^T\vec v$
    - $\dfrac{\partial L}{\partial \vec w_1^T} = (\vec n_0^T \otimes I)\vec v^T \xRightarrow{vec^{-1}}\dfrac{\partial L}{\partial W_1} = \vec n_0^T\vec v$
---
layout: post
title: Transformer 논문 정리
date: 2022-06-16 00:00:00 +0900
description: Transformer를 처음 제안한 논문 Vaswani, Ashish, et al. "Attention is all you need." Advances in neural information processing systems 30 (2017). 요약
categories: deep-learning
giscus_comments: true
---

>**요약**
>- Transformer 제안: convolution이나 recurrence가 없는 첫 sequence transduction model
>- encoder와 decoder를 각각 transformer block을 6개씩 쌓아 구현. 
>- decoder는 훈련시 역방향으로 영향을 받는 것을 막기 위한 masked multi-head attention을 transformer block에 추가
>- recurrence가 없으므로 위치에 대한 정보x → positional encoding 사용

# Abstract

- sequence transduction 모델들 중, 복잡한 Encoder-decoder with attention이 좋은 성능을 보이고 있음.
- simple한 새로운 네트워크 아키텍처, transformer를 제안. **convolution이나 recurrence가 없음!**

# Introduction

- sequence modelling / transduction 문제에서는 LSTM과 gated recurrent neural network가 state-of-the-art approach로 자리잡음
    - 그러나 recurrence의 특성상 parallelization이 불가능한 등의 문제점..
- Attention: sequence modelling / transduction model에 필수적. input/output sequence상의 거리에 관련없이 dependency를 모델링 가능
- 이 논문에서는 transformer를 제안. recurrence대신에 attention에만 기반해서 global한 input/output dependency를 만들어냄
    - parallelization 가능 → 훨씬 짧은 시간에 훈련이 가능

# Background

- 계산량을 줄이기 위한 노력으로 Extended Neural GPU, ByteNet, ConvS2S 등이 제안됨 → 모두 CNN 기반
    - 그러나 모두 input/output position 사이의 거리가 멀수록 비례해서 operation의 개수가 많아져야 함 → transformer는 operation #가 const.
- Self-attention(aka intra-attention): single sequence의 서로 다른 위치를 서로 연관지음, sequence의 representation을 계산.
- Transformer: self-attention에만 의존하여(RNN, conv 없이) input-output representation을 계산하는 첫번째 transduction model

# Model Architecture

- 높은 성능의 neural seq transduction 모델들은 모두 인코더-디코더 구조
    - encoder가 seq를 continuous representation(벡터 한 개)으로 만들고, decoder가 이를 사용해 autoregressive하게 출력 seq를 만들어냄

- Transformer stacked self-attention과 fully connected layer를 사용한 encoder/decoder로 이러한 아키텍처를 그대로 따름
- Encoder
    - Multi-head attention / Residual connection → LayerNorm → FFNN / Residual connection → LayerNorm으로 구성된 transformer block을 6개 반복
- Decoder
    - Encoder와 같은 구조에 세 번째 sub-layer를 넣음. encoder의 출력에 multi-head attention을 사용
    - encoder와 마찬가지로 Res Connection 사용
    - Masked Multi-Head Attention?
        - attention 계층은 RNN과 달리 앞뒤를 모두 볼 수 있으므로 뒤의 위치가 생성한 값들은 보지 못하게 가려줌
<p align="center" style="color:gray">
<img src="/assets/posts/2022-06-16-Transformer/스크린샷_2022-06-16_오후_5.23.23.png" width="50%" height="50%"/></p>


- Attention: query와 key-value pair. 각각 matrix로 만들어줘서 한꺼번에 계산
    
    $$
    \mathrm{Attention}(Q, K, V) = \mathrm{softmax}(\frac{QK^T}{\sqrt{d_k}})V
    $$
    
    - Q, K, V는 입력 벡터(임베딩)에 $W^Q, W^K, W^V$ 행렬 곱해서 얻음. 그냥 행렬곱이므로 dot-product attention(아마 bilinear form 안쓰고 내적해서 그렇게 부르는듯..?)
- Multi-head attention: Attention Head를 여러개 나열. 서로 다른 특성 학습
    - 각 head의 output을 concatenation → 차원이 계속 커지는걸 방지하기 위해 $W^O \in \mathbb{R}^{h d_v \times d_{model}}$ 행렬을 곱해줌
    - Transformer는 이를 세 가지 방법으로 사용.
    1. Encoder-Decoder Attention Layer: 바로 전 decoder layer에서 query 가져와, encoder output의 key/value와 함께 사용. decoder의 모든 위치에서 input seq의 모든 위치를 attend.
    2. Encoder의 attention layer: Q/K/V가 모두 이전 encoder layer의 출력에서 옴. encoder의 각 위치가 이전 layer의 모든 위치를 attend
    3. Decoder의 attention layer도 비슷
- Position-wise FFN
    - encoder/decoder가 각각 attention만 있는게 아니라 FC Feed-Forward Network도 가지고 있음. 각 **position에 각각 작용.**
    
    $$
    \mathrm{FFN}(x) = \mathrm{max}(0, xW_1+b_1)W_2 + b_2
    $$
    
- Positional Encoding: recurrence가 없기 때문에 위치에 대한 정보를 embedding에 더해줌(즉 dim이 embedding과 같음) → 그냥 sin/cos 사용
<p align="center" style="color:gray">
<img src="/assets/posts/2022-06-16-Transformer/스크린샷_2022-06-16_오후_5.51.33.png" width="40%" height="40%"/>
</p>

**Why Self-Attention**

- seq를 같은 길이의 seq로 바꿔주는 역할에서, Self-Attention과 기존 reccurence / conv layer 비교해보자!
    - layer당 계산복잡도 / 병렬화 가능한 계산량 / long-range dependency에 대한 경로 길이(당연히 짧을수록 학습이 쉬움)
        <p align="center" style="color:gray">
        <img src="/assets/posts/2022-06-16-Transformer/스크린샷_2022-06-16_오후_5.55.59.png" width="70%" height="70%"/>
        </p>
- 결론: self-attention이 계산복잡도는 높지만 나머지에서 가장 성능이 좋음
    - 계산복잡도도 n < d이면 self-attention이 더 좋음

# Training / Results

- Adam Optimizer, lr decay 적용 / Residual Dropout (p=0.1)
- English→German translation에서 기존 최고 모델보다 더 좋은 성능
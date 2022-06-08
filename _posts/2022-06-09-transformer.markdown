---
layout: custom
title: transformer 관련..
date: 2022-06-09 00:00:00 +0900
last_modified_at: 2022-06-09 00:00:00 +0900
category: docker
tags: ["nlp", "transformer"]
published: false

---
> nvidia-docker 컨테이너 실행 및 python 설치

## 1. Transformer

### 1.1. Encoder
- [https://arxiv.org/abs/1706.03762](https://arxiv.org/abs/1706.03762)

- encoder-decoder 구조
- ![Untitled](/assets/img/transformer_1.png)
- N = 6 (6개의 층)
- 각 층은 2개의 sub-layers
    - 첫 번째는 multi-head self-attention
    - 두 번째는 feed-forward
- 각 sub-layer에서는 residual connection(잔차연결)과 layer normalization 진행
- LayerNorm(x + Sublayer(x))

- residual connection(잔차연결)
    - 레이어가 깊어지면 Vanishing Gradient(기울기 소실)와 같은 문제가 발생
    - 깊어질 수록 Optimization이 어렵기 때문에 skip Connection(input x를 더함)으로 각 Layer가 학습할 정보량을 축소

- ![Untitled](/assets/img/transformer_2.png)
- 배치정규화 / 레이어정규화

- 모든 레이어는 512 차원으로 임베딩?
- esidual connection의 적용에 편의를 위해 임베딩 층을 포함한 모든 sub-layer들이 output 차원으로 d_model​=512 를 갖도록 통일했다. (residual connection시 element-wise addition을 진행하려면 차원이 같아야됨)

### 1.2. Decoder
- encoder와 동일한 두 sub-layer 사이에 encoder의 output에 대해 multi-head attention을 수행하는 sub-layer를 추가로 삽입
- 어떤 i번째 위치에서 prediction을 진행할 때, 미래의 위치들에 접근하는 것을 불가능하게하고 해당 위치 i와 그 이전의 위치들에 대해서만 의존하도록 masking 기법을 이용

### 1.3 Attention
![Untitled](/assets/img/transformer_3.png)
- 

- 참고 https://velog.io/@changdaeoh/Transformer-%EB%85%BC%EB%AC%B8%EB%A6%AC%EB%B7%B0
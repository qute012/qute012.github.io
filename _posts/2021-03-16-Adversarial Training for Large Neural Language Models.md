---
title: Adversarial Training for Large Neural Language Models
categories: [paper]
comments: true
---
microsoft research에서 발표한 논문, 간단하게 Adversarial Training을 LM에 어떻게 적용하는지 방법론을 살펴볼 예정.



**Preliminary**

1. Input Representation
   입력이 [SEP] 토큰으로 나누어진 텍스트 스팬으로 구성되어 있다고 가정함. BPE를 사용하여 subword로 나누어서 어휘 부족 문제를 해결.
2. Model Architecture
   Transformer 계열을 사용함.
3. Self Supervision
   [MASK] 토큰을 사용하는 MLM은 BERT에서 핵심적인 혁신 내용임. 그 외 BERT와 RoBERTa의 학습과정에서 Masking 규칙에 대해서 설명함.



**ALUM (Adversarial training for large neural LangUage Models)**

1. Standard Traning Objectives
   pre-training과 fine-tuning은 모두 학습 데이터에서 표준 오차를 최소화하는 것으로 볼 수 있음. 자기 지도학습 또는 지도 학습으로 부터 기반한.
   특히 학습 알고리즘은 f(x,θ): x->C,  θ로 파라미터화 된 함수 f를 학습하려고 한다. C는 task에 대해 특정한 라벨 세트이다.

**discussion**
만약 일상의 용어를 학습한 BERT를 Generalized 되었다고 가정하면, SciBert, BioBert 등의 모델들은 Specific domain에 적격한 모델이라고 볼 수 있다. 서로는 서로의 task를 잘 해결할 수 없음.

이것은 등장하는 용어의 분산 차이라고 볼 수 있는데, 사실 specific domain terms를 제외한 general terms들은 중복되는 것이 많다.



**key idea**

만약 Specific Domain BERT를 S(x)라고 가정하고, General BERT를 G(x')라고 가정할 때, x는 두 BERT의 토크나이저 차이로 각각 다른 입력이 들어가게 된다.

S(x)는 해당 도메인에 관련된 토큰일수록 많은 정보량을 가지게되고, G(x')는 General한 도메인에서 어디에 속해있지 않은 토큰이 많은 정보량을 가진다고 추정할 수 있다.

정확히 S(x)에서 각 토큰의 은닉층과 G(x')에서 각 토큰의 은닉층은 서로 다른 토큰을 가리킨다(확률적으로 같을 수도 있기는 하나 굉장히 낮은 확률일듯), 그러나 문서 레벨에서 각 은닉층을 살펴보면 정보량이 변화하는 분포를 볼 수 있다.

두 LM에서 Attention Distribution의 합은 전체 문서에 대한 distribution으로 추정할 수 있다. 

D_distribution ~~ Attention Distribution(S(x)) + Attention Distribution(G(x'))



그러나 여기서 문제점은 다른 토크나이저로 부터 생성된 시퀀스의 길이가 다르다는 것이다. 그렇기 때문에 서로 같은 t 스텝에서 각 Distribution이 보고있는 토큰은 다르게 된다.

|(S(x))| ~= |(G(x'))|



이러한 문제를 해결하기 위해, 만약 |S(x)| = k, |G(x')| = k' 라고 가정하면, k' 길이의 시퀀스에 k개의 시퀀스를 alignment 시키면 어떨까, 컨텍스트 벡터를 다음과 같이 계산하고, C' = (W(S(x))+b)* (W'(G(x'))+b')^T


$$
D'_i = \frac{exp(C'_i)}{\sum _{j=1}^nexp(C'_j)}
$$
이렇게 되면, 우리는 G(x)에 대해서 S(x)가 어디에서 매핑될 수 있는지 계산할 수 있고, 새로운 distribution을 얻을 수 있다.

그리고 새로운 출력을 아래와 같이 정의할 수 있다.

S'(x) = dot(S(x), D')
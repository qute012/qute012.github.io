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
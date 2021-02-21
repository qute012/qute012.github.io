---
title: BART Denoising Sequence-to-Sequence Pre-training for Natural Language Generation, Translation, and Comprehension
categories: [paper]
comments: true
---
Bart는 Seq-to-Seq 아키텍처를 기본적으로 삼고있다. Bidirectional Encoder(BERT)와 Autoregressive Decoder(GPT)로 구성되어 있다. 노이즈 데이터에 대해 auto encoder의 구조처럼 원래의 시퀀스로 복원하도록 학습한다.

<img src="https://user-images.githubusercontent.com/33983084/104286824-ebd29080-54f8-11eb-9108-b9657805afb1.png" alt="image" style="zoom:100%;" />

앞에 BERT와 차이점으로 두개의 Transformer의 인코더와 디코더가 연결되어 있고,  이를 reconstruction 하는 object를 가지고 있다. Self-supervised 방법론들은 NLP task에서 좋은 성능을 보인다. 기존 MLM보다 장점은 노이즈 플랙서빌리티를 제공한다.

Bert의 문제점은 independant assumption이다. 시퀀스에 corrupt된 [MASK] 토큰들은 각각 독립적인 확률을 가지고 예측한다. 그렇기 때문에 아래의 수식은 equal로 표현되지 않고, approximate로 표현된다.

![image](https://user-images.githubusercontent.com/33983084/104287725-343e7e00-54fa-11eb-851f-f06a995c3af1.png)

또한 fine-tuning에서는 [MASK] 토큰을 사용하지 않아 불일치(discrependency)가 발생한다.

Bi Transformer encoder

- decoder의 각 layer 별 cross-attention을 수행
- word prediction 전 추가적인 feed forward network(FFN)이 필요하지 않음

Autoregressive Transformers decoder

- ReLU 대신 GeLU 사용
- 파라미터를 (0, 0.02)로 초기화

noise flexibility가 있어 origin text의 변환을 적용할 수 있음



# Noise Task

1. Token Masking
   
Bert와 똑같이 80% 확률로 mask 처리, 10%는 그대로 두고, 10%는 임의의 토큰으로 변형
   
2. Token Deletion
   
토큰을 제거하여, model이 어느 자리의 토큰이 유실됐는지 결정해야 함
   
3. Text Infilling

   성능이 가장 좋은 방법, 몇 개의 토큰을 마스킹 했는지 예측하도록함
   Span Bert는 masking budget이 사용될 때까지 text span을 반복적으로 샘플링하여 토큰들을 선택하고, 각 반복마다 기하 분포를 통해 스팬의 길이를 샘플링 하도록한다. 분포는 더 짧은 스팬으로 편향되어 있음. 이후 span의 시작점을 균일하게 선택하고, 전체 샘플링된 평균 span 길이를 반환한다. probability를 0.2로 설정한 예비 시험에서 3.8의 평균 길이를 얻어 사용하였는데, Bart는 기하 분포 대신 포아송 분포( λ=3)를 사용하였고, 이 때, 가장 높은 확률을 가지는 span의 길이는 3이 된다. 결론적으로는 Masking할 때 3개의 토큰을 단일 [MASK] 토큰으로 대체할 확률이 가장 크다는 뜻이다.

![image](https://user-images.githubusercontent.com/33983084/104291872-73230280-54ff-11eb-86a5-fb73ae26847f.png)

4. Sentence Permutation
   
   Sentence를 임의의 순서로 섞어준다.
5. Document Rotation
   
   임의로 Token을 선택하고 Document를 해당 Token으로 시작하도록 Rotate

# Model Architecture

Encoder와 Decoder의 임베딩은 공유 가중치를 가지게 되고, encopder의 출력을 decoder가 다시 원래의 시퀀스로 복원하도록 구조가 되어 있음. 실제 gold와 prediction의 크로스 엔트로피 로스를 연산하여 사용

# Reference

https://arxiv.org/pdf/1910.13461

https://www.youtube.com/watch?v=VmYMnpDLPEo&ab_channel=%EB%94%A5%EB%9F%AC%EB%8B%9D%EB%85%BC%EB%AC%B8%EC%9D%BD%EA%B8%B0%EB%AA%A8%EC%9E%84

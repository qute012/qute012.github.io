---
title: Semantic Mask for Transformer based End-to-End Speech Recognition (ISCA 2020)
categories: [paper]
comments: true
---
ㅇTransformer 모델을 위한 Semantic Mask

**Abstract**

Transformer 인코더-디코더 모델은 ASR과 TTS에서 좋은 결과를 냄.

입력 시퀀스 - 출력 시퀀스의 맵핑을 사전 forced alignment없이 학습함. 학습 데이터가 적으면 과적합될 수 있어 SpecAugment를 사용하는데, 본 논문에서는 일반화된 의미론적 마스키 방법을 제안한다.



**Semantic Masking**

1. **Masking Strategy**
   본 기법을 쓰기 위해서는 강제 정렬을 필요로함. 여기서는 Montreal forced aligner를 사용함, 단어 레벨에서 시간 정보를 포함한. 이 정보로 부터 임의의 토큰을 masking하는데, 기존 SpecAugment에서는 실제 발화가 아니여도 masking 할 수 있는데, 본 논문에서는 실제 발화만을 마스킹하도록 함, 15%의 샘플을 전체 발화의 평균값으로 마스킹함 => 이 부분은 곧 SpecAugment의 주요 마스킹 기법을 적용할 수 있음.
2. **Why Semantic Mask Works?**
   SpecAugment는 과적합을 방지하기 위해 입력을 손상시켜 음향 모델의 과적합을 방지하는데, 반면에 본 모델은 디코더가 언어 모델을 더 잘 배울수 있도록 함. 이것은 곧 E2E(End-to-End) 모델에서 디코더가 마스킹 되지않은 signal에서 마스킹된 단어를 예측하는데 기반되게 함. 기존 방식에서 가장 큰 개선점인 듯 하다. 디코더가 alignment를 하고 단어를 예측할 때, masking된 부분 역시 하나의 캐릭터로 예측될 수 있는데, 실제 그 부분은 발화를 하지 않는 부분일 확률이 있다. 이는 곧 연관이있는 음성만을 참고하여서 단어를 생성하도록 돕는다. 이는 노이즈가 있을 때 더 효과적이라고 함.



**Performance** 

테이블 중간을 보면 SpecAugment와 비교가 있는데, other dataset을 보면 성능 향상이 엄청나다.

![image](https://user-images.githubusercontent.com/33983084/110899573-af7aaf80-8344-11eb-9a93-d28aa14d71a0.png)

또 다른 실험으로는 LM과 함께 쓸 때를 ablation test 하였는데, LM과의 결합에서는 특히 더 좋은 성능향상을 보여준다.

![image](https://user-images.githubusercontent.com/33983084/110899732-fa94c280-8344-11eb-9b41-7affca23a443.png)

영어권 음성인식은 이미 높은 성능을 도달하였고, 개선된 기법들의 성능향상이 크게 일어나는 것 같지 않아보이지만, 한국어에서는 LAS에서 Transformer 그리고 더 개선된 모델이 나올수록 엄청난 성능향상을 보여준다. WER 성능 차이가 엄청남. . . .



**Disscution (내생각)**

본 실험에서는 몬트리울 aligner를 사용하였는데, 이 역시 Transformer가 학습되면서 함께 학습되도록 개선할 수 없을까...? 그렇게되면 Transformer의 Alignment 역시 더 올바른 학습이 가능할 것으로 보일듯...

아마도 이러한 masking 기법에서의 성능향상도 있으면서 준지도학습의 alignment 학습의 효과도 있어보임.
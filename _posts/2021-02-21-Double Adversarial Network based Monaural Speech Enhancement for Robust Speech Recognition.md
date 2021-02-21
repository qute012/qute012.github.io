---
title: Double Adversarial Network based Monaural Speech Enhancement for Robust Speech Recognition
categories: [paper]
comments: true
---
## Double Adversarial Network based Monaural Speech Enhancement for Robust Speech Recognition
본 논문에서 제안하는 모델의 구조는 아래의 그림과 같다.

![image](https://user-images.githubusercontent.com/33983084/108619929-954b6100-746b-11eb-8e67-9198449b4ac3.png) 

기존 SEGAN과 비교할 때, Generator가 생성한 fake speech를 Discriminator에 넣지 않고, Enhance 모델과 Generator 모델 두 개가 적대적으로 학습된다. 이 때, SEGAN에서는 Generator의 입력으로 노이즈 인풋이 들어가는데, 본 논문에서는 표준 정규분포에서 추출된 0~1의 값들로 이루어진 랜덤한 벡터 z를 사용한다고 한다. Ablation Study에서 각 요소를 제거하면서 성능을 비교할 때, 제안한 모델인 DAN이 가장 높은 성능을 보였음.

Enhanced Speech의 오차 계산은 평균 제곱 오차를 사용하고, Generator는 아래에 수식과 같이 평균 제곱근 편차를 계산하여 사용한다.

![image](https://user-images.githubusercontent.com/33983084/108620034-76010380-746c-11eb-897e-987e805d5500.png)

이러한 구조를 사용할 때, 추측으로 성능이 개선되는 이유는 임의의 벡터 z를 Clean Speech와 옵티마이즈 하면서 Discriminator가 노이즈에 대한 식별에 강인함이 높아지는 걸로 추측된다.



# Reference

https://isca-speech.org/archive/Interspeech_2020/pdfs/1504.pdf

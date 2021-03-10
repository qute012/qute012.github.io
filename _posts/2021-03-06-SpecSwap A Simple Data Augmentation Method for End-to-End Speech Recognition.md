---
title: SpecSwap A Simple Data Augmentation Method for End-to-End Speech Recognition (ISCA 2020)
categories: [paper]
comments: true
---
Augmentation 아이디어를 얻기위해 방법론만 간단히 살펴보았음.

**SpecSwap Policy**

입력 시퀀스에 직접 작용하는 어그멘테이션 정책을 목표로 하였음. 이 정책은 log-mel 스펙트럼 인코드의 일반화를 개선할 수 있음: 주파수 교체(frequency swapping)와 시간 교체(time swapping)

사진을 보면 바로 어떤 기법인지 알 수 있따. 특정 주파수의 두 영역을 교체, 특정 시간대의 두 영역을 교체

![image](https://user-images.githubusercontent.com/33983084/110627189-fd78a180-81e4-11eb-80c3-923ff3216035.png)

파라미터는 SpecAugment와 유사함. 임의의 변수 f의 2배만큼 주파수 영역 선택, 임의의 변수 t의 2배만큼 시간 영역 선택.

![image](https://user-images.githubusercontent.com/33983084/110627423-4f212c00-81e5-11eb-824f-131d0c9ff2dc.png)

두가지 파라미터를 모두 쓸 때, 약간의 성능 향상이 있었음.

또한 SpecAugment, SpecSwap, Speed Perturb 각각과 모두 쓸 때의 성능 비교도 하였는데, 모두 쓸 때 모델이 로버스틱함.

![image](https://user-images.githubusercontent.com/33983084/110627525-711aae80-81e5-11eb-8d9f-07622b1adb05.png)
---
title: Categorical Reparameterization with Gumbel-Softmax (ICLR 2017)
categories: [paper]
comments: true
---
**Abstract**

범주형 변수는 현실에서 이산화 구조를 표현하는 자연스러운 방법.

그러나 확률적 뉴럴 네트워크는 각 샘플의 역 전파를 할 수 없기 때문에 범주형 잠재 변수를 드물게 사용한다.

본 논문에서는 범주형 분포의 미분 불가능한 샘플을 새로운 gumbel-softmax 분포의 미분 가능한 샘플로 대체하는 효율적인 기울기 추정기를 제시.



**Contribution**

1.  gumbel-softmax를 소개하고, 일반적인 차원의 연속적인 분포를 범주형 샘플에 근사할 수 있는, reparameterization trick을 사용하여 쉽게 파라미터의 기울기를 계산할 수 있다.
2.  gumbel-softmax는 베르누이 변수와 범주형 변수들에서 단일 기울기 추정기보다 더 좋은 성능을 실험적으로 보임.
3.  추정기가 관측되지 않은 범주형 잠재 변수에 대한 확률을 구하는데 비용이 들지 않고 준지도 학습 모델을 학습할 수 있다.



**Gumbel-Softmax Distribution**

gumbel-max trick은 연속 균등분포에서 u를 뽑아 엡실론을 더하고 -log를 취하고, 결과값에 엡실론을 더하고 로그를 한번 더 취함, 이 값을 g라고 하고 로지스틱값에 g를 더해서 가장 큰 인덱스로 원-핫 인코딩을 함.

![image](https://user-images.githubusercontent.com/33983084/110621329-6bb96600-81dd-11eb-871d-91ef7ec3601b.png)

그러나 여기서 argmax은 비분이 불가능함.



gumbel-softmax은 argmax를 softmax로 바꾸어서 미분을 가능케함. 

![image](https://user-images.githubusercontent.com/33983084/110623330-0c108a00-81e0-11eb-9214-fed8ea9c87f1.png)

여기서 τ은 temporature parameter로

0에 가까워지면 softmax의 결과 중 가장 큰 값이 커지면서 원-핫 인코딩 구조의 이산화가 되고, 무한대로가면 연속 균등분포를 갖는다. τ의 값은 얼만큼 gumbel-softmax 분산을 스무딩 하는지에 대한 값이며, 이 분산은 파라미터   π와 관련하여 잘 정의된 기울기가 존재한다.



Wav2Vec 2.0에서 quantization 모듈로 gumbel-softmax를 사용하였는데, 음성의 연속적인 representation을 매우 작은 temporature parameter를 사용하여 원-핫 인코딩 한다. 라벨 역시 동일한 방식으로 quantization되는데, 음성의 컨텍스트 표현에서 gumbel-softmax의 원-핫 인코딩으로 표현된 output과 label이 얼마나 유사한지를 목적으로  pretrained encoder를 학습한다.
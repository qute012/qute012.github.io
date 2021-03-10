---
title: Fast and Slow Acoustic Model (ISCA 2020)
categories: [paper]
comments: true
---
Fast AM과 Slow AM이 결합된 AM을 제안함. Slow AM은 정확도가 높고, 모델의 크기가 확실히 더큼.

Fast AM은 일반적으로 애플리케이션을 위해 개발됨. Fast AM은 더 빠른 latency를 유도하지만 약함.

Fast AM에서 출력 상태 정보를 인코딩하고, Slow AM을 사용하여 전체적인 모델 정확도를 향상시키도록 함.

두 개를 결합하기 위해 Fast AM은 3-4배 작게 만들고, 두 개의 Slow AM으로 성능을 개선하도록함.

![image](https://user-images.githubusercontent.com/33983084/110629201-40d40f80-81e7-11eb-92dd-4f09dce59d0b.png)

대략적인 구조는 위와 같고, 자세한 구조는 아래의 그림을 따른다. 약간 transducer 같이 생김.

![image](https://user-images.githubusercontent.com/33983084/110629270-547f7600-81e7-11eb-9550-8ecec0dfa50c.png)

입력으로부터 forward되는 과정은 그림을 보면 이해하기 쉽다. 그러면 각 인코더가 어떻게 구성되어있는지 보자.

먼저 Slow AM은 매우 깊게 설계되었고, Fast AM은 성능은 최소한으로 빠르게 설계되었음, 예를들어 크기, 연산 또는 배터리를 게이팅하는것.



전체 모델의 레이어 구조는 아래와 같음, 순서대로 Slow Encoder/Fast AM/ Fast Encoder/Joint Encoder and Softmax

![image](https://user-images.githubusercontent.com/33983084/110630464-b8566e80-81e8-11eb-951d-e3209f464dc8.png)

**Slow Encoder**

6개의 은닉층을 가진 전형적인 AM의 경우 초기 4개의 레이어로 구성될 수 있다.

Slow encoder는 Bi LSTM 또는 Uni LSTM으로 구성된 첫번째 레이어 또는 바람직한 레이어(? 시퀀스를 인코딩하기 때문에 RNN 계열 GRU 등 또는 트랜스 포머 계열인 듯 함)가 될 수 있음.



**Fast AM**

Softmax, Logits or hidden layer states로 표현할 수 있음. 대략적으로 아래의 구조 중 하나일 것 같음.

x^ = Wx+b

1) Softmax(x^)	2) σ (x^)	3) x^



 **Fast Encoder**

Fast Encoder는 Slow Encoder에 상응하는 Fast AM 출력을 생성함. Fast AM과 Slow Encoder는 다른 벡터 공간임(서로 시간 정보가 다름=>Slow Encoder의 LSTM의 레이어의 개수 l 에 따라서 각 히든 스테이트는 약 l만큼 지연됨). Fast AM 출력이 Slow AM보다 3-4개의 은닉층을 앞서있는 점에 주목했다. 이러한 이점에서 현재 Fast AM의 output at는 at+1...at+k(k는 미래 은닉층)에서 관측됨. 이러한 Fast AM의 출력 상태는 핵심적인 혁신이라고 한다!! => 은닉층에서 짧아지는 시간을 고려하여 Fast AM의 출력과 Slow Encoder의 출력을 얼라이먼트가 가능하다는 것을 뜻하는 듯 함.



**Joint Encoder and Softmax**

두 인코더의 정보를 결합, attention을 사용하여 결합이 가능하기도 하지만 여기서는 단순하고 최소한의 매개 변수를 위한 연산을 함(?). 그건 바로 Softmax 함수!  결합하고 Softmax를 하는데, 음성 상태의 사후 상태를 생성하기 위해 디코더 레이어를 사용함.  hidden dim*2가 입력 차원인 fc레이어를 사용하여 결합된 정보를 softmax함.

음성의 사후 상태는 DNN-HMM 구조인데 이부분은 따로 정리하지 않겠음.



속도 성능이 많이 개선된다고함
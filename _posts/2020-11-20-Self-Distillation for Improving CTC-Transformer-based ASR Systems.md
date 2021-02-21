---
title: Double Adversarial Network based Monaural Speech Enhancement for Robust Speech Recognition  (ISCA 2020)
categories: [paper]
comments: true
---
Interspeech 2020에 개제된 논문이다.

# Abstract

Sequence-to-sequence(S2S) 모델은 안정적으로 ASR 시스템에 정착되었고, 핵심 요소는 attention 메커니즘이 입력과 아웃풋인 캐릭터를 정렬(alignment)하는 것이다. 어텐션의 가중치는 입력과 출력의 시퀀스의 타임 프레임을 알려준다. 본 연구에서는 매 시점의 트랜스포머의 출력과 어텐션의 가중치를 사용하여 pseudo-targets을 생성하고,  CTC-Transformer의 shared encoder를 Transformer-decoder를 통해 피드백 받아 더 많은 정보를 학습한다. 증류 기법은 흔하지만, 이를 디코더로 부터 피드백을 받는 방법에 대해서는 감이 안와서 method를 더 세밀하게 분석할 필요가 있어보인다. 이런 방식으로 모델을 학습하면 S2S 역시 공유 인코더로 인해서 성능이 강화된다고 한다.

# ASR Modules

baseline은 Transformer와 CTC이다. Transformer의 output은 문장의 끝을 나타내는 <eos>와 <pad> 또는 <blank> 토큰을 포함한다. 

#### Transformer

multi-head- attention (MHA)

트랜스포머는 인코더와 디코더 블록으로 구성되있다. 각 블록은 scale dot-product attention을 가지고 있음. 최근 등장한 모델은 대부분 attention 메커니즘을 사용하기 때문에 여기서 구현과 함께 공부해보도록 한다.

![image](https://user-images.githubusercontent.com/33983084/103991741-74d68880-51d6-11eb-8584-4461d36f7609.png)

```
class ScaledDotProductAttention(nn.Module):

    def forward(self, query, key, value, mask=None):
        dk = query.size()[-1]
        scores = query.matmul(key.transpose(-2, -1)) / math.sqrt(dk)
        if mask is not None:
            scores = scores.masked_fill(mask == 0, -1e9)
        attention = F.softmax(scores, dim=-1)
        return attention.matmul(value)
```

Attention의 내부 구조는 query와 key의 transpose를 내적하고, softmax를 취한후 attention score를 구하고 value와 곱하여 context를 구하는 연산이다. multi-head attention은 각각의 Q, K, V를 head 개수만큼 분리하여 각각을 따로 연산한다. 보통 attention을 사용할 때, 파라미터로 Q, K, V 그리고 mask를 받게 된다. mask는 가중치 값을 -inf와 같이 매우 큰 음수 값으로 보내버려서 softmax에서 0을 얻게 한다. Transformer에서는 square subsequent mask와 key padding mask가 있다. 두개의 차이점은 시퀀스에 대응을 하느냐 아니면 batch에 대응을 하느냐이다.

```
class MultiHeadAttention(nn.Module):

    def __init__(self,
                 in_features,
                 head_num,
                 bias=True,
                 activation=F.relu):
        """Multi-head attention.
        :param in_features: Size of each input sample.
        :param head_num: Number of heads.
        :param bias: Whether to use the bias term.
        :param activation: The activation after each linear transformation.
        """
        super(MultiHeadAttention, self).__init__()
        if in_features % head_num != 0:
            raise ValueError('`in_features`({}) should be divisible by `head_num`({})'.format(in_features, head_num))
        self.in_features = in_features
        self.head_num = head_num
        self.activation = activation
        self.bias = bias
        self.linear_q = nn.Linear(in_features, in_features, bias)
        self.linear_k = nn.Linear(in_features, in_features, bias)
        self.linear_v = nn.Linear(in_features, in_features, bias)
        self.linear_o = nn.Linear(in_features, in_features, bias)

    def forward(self, q, k, v, mask=None):
        q, k, v = self.linear_q(q), self.linear_k(k), self.linear_v(v)
        if self.activation is not None:
            q = self.activation(q)
            k = self.activation(k)
            v = self.activation(v)

        q = self._reshape_to_batches(q)
        k = self._reshape_to_batches(k)
        v = self._reshape_to_batches(v)
        if mask is not None:
            mask = mask.repeat(self.head_num, 1, 1)
        y = ScaledDotProductAttention()(q, k, v, mask)
        y = self._reshape_from_batches(y)

        y = self.linear_o(y)
        if self.activation is not None:
            y = self.activation(y)
        return y

    @staticmethod
    def gen_history_mask(x):
        """Generate the mask that only uses history data.
        :param x: Input tensor.
        :return: The mask.
        """
        batch_size, seq_len, _ = x.size()
        return torch.tril(torch.ones(seq_len, seq_len)).view(1, seq_len, seq_len).repeat(batch_size, 1, 1)

    def _reshape_to_batches(self, x):
        batch_size, seq_len, in_feature = x.size()
        sub_dim = in_feature // self.head_num
        return x.reshape(batch_size, seq_len, self.head_num, sub_dim)\
                .permute(0, 2, 1, 3)\
                .reshape(batch_size * self.head_num, seq_len, sub_dim)

    def _reshape_from_batches(self, x):
        batch_size, seq_len, in_feature = x.size()
        batch_size //= self.head_num
        out_dim = in_feature * self.head_num
        return x.reshape(batch_size, self.head_num, seq_len, in_feature)\
                .permute(0, 2, 1, 3)\
                .reshape(batch_size, seq_len, out_dim)

    def extra_repr(self):
        return 'in_features={}, head_num={}, bias={}, activation={}'.format(
            self.in_features, self.head_num, self.bias, self.activation,
        )
```

입력된 Q, K, V와 동일한 가중치 매트릭스를 각각 연산하고, head의 개수만큼 벡터를 조각낸다. 그 후 위에서 attention 연산을 그대로 적용하면 각 헤드끼리 연산이 되고, 최종적으로 다시 head를 하나의 벡터로 선형 결합하는 구조이다.



Speech encoder of Transformer

가장 초기의 입력을 Conv2D로 피처 맵을 형성하고 linear embedding 한다. 이 과정을 subsampling이라고 부르는데 피처 맵의 압축이라고 이해하였다.

![image](https://user-images.githubusercontent.com/33983084/103995297-a43bc400-51db-11eb-9741-8a8acebcbdaa.png)

임베딩된 결과는 각 time step에 따라서 위치적으로 인코딩 되고, positional encoding(PE)은 Wav2Vec 2.0과 다르게 sinusoidal positional encoding을 사용하였고, 임베딩 된 각각의 time step에 따라서 아래의 수식과 같이 위치 인코딩 된다.  이에 대한 설명과 구현은 Positional Encoder으로 따로 정리를 해야겠다.

![image](https://user-images.githubusercontent.com/33983084/103995964-86bb2a00-51dc-11eb-9262-4c1640780933.png)

```
class PositionalEmbedding(nn.Module):

    def __init__(self, d_model, max_len=512):
        super().__init__()

        # Compute the positional encodings once in log space.
        pe = torch.zeros(max_len, d_model).float()
        pe.require_grad = False

        position = torch.arange(0, max_len).float().unsqueeze(1)
        div_term = (torch.arange(0, d_model, 2).float() * -(math.log(10000.0) / d_model)).exp()

        pe[:, 0::2] = torch.sin(position * div_term)
        pe[:, 1::2] = torch.cos(position * div_term)

        pe = pe.unsqueeze(0)
        self.register_buffer('pe', pe)

    def forward(self, x):
        return self.pe[:, :x.size(1)]
```


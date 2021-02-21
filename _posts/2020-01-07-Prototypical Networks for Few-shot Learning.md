---
title: Prototypical Networks for Few-shot Learning
categories: [paper]
comments: true
---
Features Extraction 부분은 데이터마다 상이하므로 넘어가고, Data를 load하는 방법과 모델의 학습 구조 위주로 공부

DataLoader에서 하나의 mini-batch를 만들때, Sampler를 사용하여 각 클래스마다 Supporset을 만듬
```
def __iter__(self):
        for i_batch in range(self.n_batch):
            batch = []
            classes = torch.randperm(len(self.m_ind))[:self.n_cls]
            for c in classes:
                l = self.m_ind[c]
                pos = torch.randperm(len(l))[:self.n_per]
                batch.append(l[pos])
            batch = torch.stack(batch).t().reshape(-1)
            yield batch
```

모델의 구조는 Decoder는 없이 Encoding만 implement를 하는데 이는 쿼리와 서포트 셋에 대해 합성곱을 수행한 후 인코딩된 쿼리 벡터와 서포트 벡터와의 유클리디안 거리가 가장 가까운 클래스로 분류한다.
```
class Convnet(nn.Module):

    def __init__(self, x_dim=3, hid_dim=64, z_dim=64):
        super().__init__()
        self.encoder = nn.Sequential(
            conv_block(x_dim, hid_dim),
            conv_block(hid_dim, hid_dim),
            conv_block(hid_dim, hid_dim),
            conv_block(hid_dim, z_dim),
        )
        self.out_channels = 1600

    def forward(self, x):
        x = self.encoder(x)
        return x.view(x.size(0), -1)
```

벡터의 유클리디안 거리를 구하는 공식은 아래와 같다.

```
def euclidean_metric(a, b):
    n = a.shape[0]
    m = b.shape[0]
    a = a.unsqueeze(1).expand(n, m, -1)
    b = b.unsqueeze(0).expand(n, m, -1)
    logits = -((a - b)**2).sum(dim=2)
    return logits
``` 
두 벡터 사이의 공간거리를 계산하여 각 클래스에 대한 로지스틱 값을 계산하고 그에 대한 엔트로피를 계산한다.

# Reference
https://github.com/cyvius96/prototypical-network-pytorch

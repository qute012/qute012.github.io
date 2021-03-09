---
title: NamedKeys Unsupervised Keyphrase Extraction for Biomedical Documents (ACM 2019)
categories: [paper]
comments: true
---
**Abstract**

문서를 처리하는 다양한 모델들이 제안되었지만, 자동화된 키워드 추출은 다른 자연어 처리 작업과 비교하여 성능이 떨어짐, 특히 생물 의학 분야에서는 훨씬 더 해결하기 벅차다.

이러한 문제를 해결하기 위해 NamedKeys를 사용해서 생물 의학 텍스트에서 의미 있고 유익한 키 문구를 자동으로 식별,  그리고 이러한 데이터 셋을 제안함.



**Method**

**Candidate Keyphrase Generation**

1. 개체명을 인식하여 keyphrase의 후보군으로 선정
2. 단어 레벨의 phrases를 추출하여 후보군으로 선정
3. 문서 가중치(D)와 1, 2의 임베딩 점수를 더함
4. page rank 알고리즘을 적용하여 semantic한 키프레이즈를 뽑아냄

3번 단계부터 살펴보면, 임베딩 점수는  Word2Vec이나 FastText를 적용한다고 함. Chunking으로 부터 나온 keyphrases와 NER로 부터나온 keyphrases의 사전 구축된 임베딩으로 부터 점수를 구함.

문서 D의 representation은 아래의 수식으로 근사한다.

![image-20210309174943993](C:\Users\qute0\AppData\Roaming\Typora\typora-user-images\image-20210309174943993.png)

해당 토큰(여기서는 keyphrase)이 전체 문서에서 등장한 확률의 역에 log를 취한 IDF를 구하고, wi는 named entity 에 상응하는 벡터이다. 

최종적으로 문서(D)와 각 keyphrase representation의 코사인 유사도(cosine similarity)를 구함.

![image-20210309175913173](C:\Users\qute0\AppData\Roaming\Typora\typora-user-images\image-20210309175913173.png)

**Phrase Quality**

앞에 과정에서 구한 keyphrase들이 코사인 유사도가 높을 수 있지만, 아직 구문적으로 의미 있는 구라고 볼 수 없음. 예를 들어서 "‘ventricular arrhythmias vary"와 "“ventricular arrhythmias"은 코사인 유사성이 높을 수 있지만, 구문적으로 좋아보이지 않는다.

본 논문에서 keyphrase 후보군들의 품질을 올리기 위해 새로운 랭킹 알고리즘인 Information Frequency를 제안한다. 점별 상호정보량(pointwise mutual information)이란 개념을 적용하였다.

![image-20210309182418953](C:\Users\qute0\AppData\Roaming\Typora\typora-user-images\image-20210309182418953.png)

pmi의 수식으로 부터, x,y 단어의 빈도수를 곱해주면서 x,y가 동시에 일어나면서 이 문서에서 얼만큼 자주 등장하는지에 대한 정보량을 계산한다.

![image-20210309180518129](C:\Users\qute0\AppData\Roaming\Typora\typora-user-images\image-20210309180518129.png)

각 파라미터들을 정리해보면,

p(x,y): 두개의 단어가 함께 존재할 수 있는 확률

p(x):  텍스트에서 첫번째 단어일 확률

p(y): 텍스트에서 두번째 단어일 확률

freq(x,y): 두개의 단어 등장하는 횟수

이러한 방법은 pmi가 항상 각 단어의 발생 확률에 상대적인 동시 발생 확률을 계산하는데, 각 단어의 빈도수가 많아질수록 pmi 점수는 작아진다. 여기서는 구 빈도와 구성 단어의 빈도를 기반으로 pmi 수식을 개선하여 단어의 표현성을 측정한다. 해당 단어가 텍스트에서 첫번째, 두번째에 존재하지 않을 수록,  두 단어가 함께 존재할 수록 더 많은 정보량을 가지며, 두 단어의 빈도수를 곱해서 정보량을 계산한다.

![image-20210309182156990](C:\Users\qute0\AppData\Roaming\Typora\typora-user-images\image-20210309182156990.png)

위의 예시에서 베이스라인에서 high risk prostate cancer은 높은 정보량을 가지는 것으로 되어있는데, 이 어구자체가 문서와의 유사도가 높다고 판별된다. 하지만 high/risk/prostate/cancer 각각의 단어를 보면 prostate와 cancer은 본문에서 함께 표현될 가능성이 높다. 하지만 high와 risk는 prostate와 cancer과 함께 표현될 확률이 낮다. 그렇기에 개선된 방법에서는prostate cancer만을 핵심어구로 보게된다. 

여기서는 랭킹알고리즘(페이지 랭킹에 기반한 수식 같음)을 사용한다.

![image-20210309190632322](C:\Users\qute0\AppData\Roaming\Typora\typora-user-images\image-20210309190632322.png)

임의의 알파값을 설정하여, 이 알파값보다 높은 랭킹 점수를 가진 어구를 핵심어구라 한다. 본 논문에서는 0.75로 사용하였음. 이 부분은 깊게 보지 않았음.



**Dataset**

본 논문에서는 PubMed Central Open Access Subset articles에서 저자가 사용한 최소 5개 이상의 키워드를 핵심어구로 사용하였으며, 데이터 셋이 제공되는 내용은 아래와 같음. 2700만개의 기사중에 3049개만이 기사 제목 서론 5개 이상의 핵심어구를 가지고 있음. 데이터 셋을 다운하는 곳은 https://github.com/ZelalemGero/NamedKeys 여기서 다운할 수 있음.

![image-20210309191412034](C:\Users\qute0\AppData\Roaming\Typora\typora-user-images\image-20210309191412034.png)

감쇠 계수가 뭔지 모르지만 0.85로 사용했다고 함...  측정 방법은 F1 score를 사용하였음.



**Performance**

결과는 아래 표와 같음.

![image-20210309191620680](C:\Users\qute0\AppData\Roaming\Typora\typora-user-images\image-20210309191620680.png)
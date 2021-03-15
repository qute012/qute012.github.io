---
title: Key2Vec Automatic Ranked Keyphrase Extraction from Scientific Articles using Phrase Embeddings (ACL 2018)
categories: [paper]
comments: true
---
**Abstract**

keyphrase 추출은 자연어 처리에 기본이 되는 task이고, 문서를 상징하는 phrase들을 결정한다.

본 논문에서는 과학 문서에서 추출된 어구의 랭킹을 구하기위해 어구에 이점이 있는 임베딩의 비교사 학습 기법(Key2Vec)을 소개한다. 특히 주제 표현과 PageRank를 사용한 주요 구문 순위 알고리즘과 multi-word phrase 임베딩을 학습하기 위한 텍스트 문서의 처리의 효율적인 방법을 제안함.



**Method**

주요 방법론의 순서는 후보 선택, 후보 점수, 후보키 랭킹이다. 모든 단계는 텍스트 처리와 phrase 임베딩 모델에 의존적임. 대량의 과학문서를 학습 시킴.



1. **Text Processing**
   unigram 단어와 혼합된 다중 단어 구문이 존재하면 Word2Vec과 같은 기술을 사용하여 훈련된 임베딩 모델의 성능과 정확도가 증가함. 그러나 본 논문에서는 의미있고 응축된 어구의 chunk를 뽑아내기위해 다른 방법으로 접근함. Word2Vec과 같이 단어가 함께 존재할 확률에서 접근하는 모델이 아닌 Named Entity Extraction model을 사용함.

   ![image](https://user-images.githubusercontent.com/33983084/111181121-fc82ae00-85f0-11eb-8e06-c76e7744ea37.png)

   위에 구조와 같이 Named Entity로 PROPN(proper noun, 특정 개인, 장소 또는 객체 또는 이름의 일부인 명사)
   를 추출하고 chunking을 사용해서 명사 어구를 추출한다. Spacy라는 라이브러리를 사용하였다고 함.

   추출한 후보키들중 single word와 multi-word는 아래 방법으로 정제하였다고 한다.

   > • Noun phrases and named entities that are fully numeric. 
   >
   > • Named entities that belong to the following categories are filtered out : DATE, TIME, PERCENT, MONEY, QUANTITY, ORDINAL, CARDINAL. Refer, Spacy’s named entity documentation2 for details of the tags. 
   >
   > • Standard stopwords are removed. 
   >
   > • Punctuations are removed except ‘-’.

   * 명사구와 엔터티가 숫자인 경우
   * 엔터티가 다음 카테고리에 들어가는 경우
   * stopwords인 경우
   * -를 제외한 구두점은 제외

   

   multi-word 명사구의 앞 뒤 토큰을 아래 과정으로 정리한다고 함.

   > • Common adjectives and reporting verbs are removed if they occur as the first or last token of a noun phrase/named entity. 
   >
   > • Determiners are removed from the first token of a noun phrase/named entity. 
   >
   > • First or last tokens of noun phrases/named entities belonging to following parts of speech: INTJ Interjection, AUX Auxiliary, CCONJ Coordinating Conjunction, ADP Adposition, DET Interjection, NUM Numeral, PART Particle, PRON Pronoun, SCONJ Subordinating Conjunction, PUNCT Punctutation, SYM Symbol, X Other, are removed. For a detailed reference of each of these POS tags please refer Spacy’s documentation3 . 
   >
   > • Starting and ending tokens of a noun phrase/named entity is removed if they belong to a standard list of english stopwords and functional words.
   > 

   * 일반 형용사(good, new, first 등)와 보고 동사(설명을 위한 동사)가 명사구/엔터티의 첫번째 또는 마지막 토큰일 경우
   * 결정자(my, your 등)는 명사구/엔터티의 첫번째 토큰으로 부터 제거됨
   * 무의미한 접속사들을 얘기하는 듯 함
   * 명사구/엔터티 앞뒤의 stopword

   외에도 괄호, 공백, '(아포스트로피) 등 불필요한 문자들을 제거함

2. **Training Phrase Embedding Model**
   임베딩 모델의 목적은 단일 단어와 다중 단어 구문으로 구성된 텍스트 단위 사이의 의미적 및 구문적 유사성을 포착하는 것이다. 본 논문에서는 FastText를 사용하였는데, 단어 사이의 의미론적 유사성과 형태소 유사성 모두 포착하기 때문. <u>이러한 관점에서 Bert는 단어를 더 의미있는 sub word로 나누었음</u>. 하나의 단어를 여러 토큰으로 분리하여 단어 사이의 관계를 정의할 수 있음.
   데이터 셋은 다양한 분야의 1,147,000개의 과학 초록을 사용하였음. arXiv API에서 제공된다고함.

3. **Candidate Selection**
   이 스텝에서는 문서로부터 추출될수 있는 전체 후보군에서 keyphrase의 후보군들을 선정하는 것을 목표로 하였음. 본 스텝에서는 문서에 대한 고유한 n개의 후보키가 남게된다. 방법은 2.1에서 선정된 각각 다른 휴리스틱 방법들을 사용함

   ![image](https://user-images.githubusercontent.com/33983084/111184657-83855580-85f4-11eb-84e8-a373ca45df6f.png)

   

4. **Candidate Scroing**

   theme vector(τˆd_i )와 document(d_i)변수가 나옴. theme vector는 문서의 유형과 얻고자 하는 최종 keyphrase들에 따라 조정됨. 본 논문에 실험으로 사용된 Inspec에서는 첫번째 문장, SemEval에서는 타이틀과 처음 10개의 문장을 추출함. Phrase Embedding 모델에 넣고, 모든 벡터의 합을 구해서 theme vector를 만듬. 또한 각 후보키에 대해서 벡터(candidate keyphrase)도 구해놓음.

   후보키와 에 대한 cosine distance를 구해서 점수를 구할 수 있음, 1일 경우에 완전히 주제와 밀접한 관계를 가진 키, 0일 경우에는 완전히 상관없는 키. 최종적으로 thematic weight를 구할 수 있음. 

   ![image](https://user-images.githubusercontent.com/33983084/111186416-58036a80-85f6-11eb-8e29-4a0261a4f2f7.png)

   각 문서 별 후보키들에 대한 theme vector와의 유사성을 포함하는 weight이다.

5. **Candidate Ranking**
   후보키들에 대해서 마지막 랭킹을 내기위해 PageRank 알고리즘을 사용함. 본 알고리즘에서 directed graph  G_di는 주어준 문서의 후보키 C_di들을 정점으로하고 E_di는 두 후보키를 연결하는 가장자리가 5의 window 크기 내에서 동시에 발생하는 경우 가장자리를 연결함. 각 정점 c^di_j와 간선 E(c^di_j)로 이루어진 그래프 G가 생김,  c^di_j에는 다른 정점으로 이어지는 확률이 담겨있음 => coocur(c_i, c_j)/(Word의 개수/5)

   ![image](https://user-images.githubusercontent.com/33983084/111187253-2e970e80-85f7-11eb-9146-55980163a543.png)

   4에서 얻은 후보키 구문과 동시 발생 빈도 사이의 의미적 semantic similarity(sr)을 사용하여 가장자리에 대해 계산된다. SR은 유사성이 높을수록 의미론적 정보량은 적다고 가정함

   ![image](https://user-images.githubusercontent.com/33983084/111188072-fcd27780-85f7-11eb-96c6-bf21cb24832c.png)

   샤논의 법칙에서 확률이 높을수록 정보량은 낮아진다는 점에서 위에 수식을 근사한 듯 함. 유사할수록 얻을 수 있는 정보량은 적어진다고 가정.

   ![image](https://user-images.githubusercontent.com/33983084/111188613-96018e00-85f8-11eb-8b3a-f9ce255efccf.png)

   최종적인 sr은 위에 수식으로 구하게 되는데, semantic이 의미론적 정보이고, coocur은 동시에 일어날 확률이다.  만약 두 변수가 종속적일 경우 확률은 P(A)*P(B)가 되는데, 현실적인 관점에서 단어들은 사실 어느정도 종속적인  확률을 가진다고 생각하지만, 그 경우의 수가 너무 많아서 독립적으로 두는듯함. 이럴 때 사용하는 방법이 Pointwise Mutual Information(PMI)인 점별 상호정보량이라는 독립사건 a,b가 동시에 일어날 확률을 구한다. PMI를 조금 알아보면 아래 수식과 같이 계산된다. x,y 두 사건이 같이 많이 일어나면 두 사건은 연관성이 있다고 하는게 조건부 확률인데, 여기에 각자 독립확률을 나눠주어서 각자 독립된 확률이 작을수록 더 높은 정보량을 가지고 있게만듬. log는 각 정보량을 계산할 때 덧셈연산이 가능하도록 취함(엔트로피 계산식 유도할 때 나옴)

   ![image](https://user-images.githubusercontent.com/33983084/111189217-36f04900-85f9-11eb-9507-e5e840669974.png)

   두 사건이 동시에 일어날 확률을 각각 일어날 확률로 나눠준 것. 이것을 의미있게 해석을하면, 문서에 대해 후보키들이 동시에 발생을 하지만 따로는 존재하지 않을수록 더 높은 정보량을 가지고 있다고 결론낼 수 있다.

   최종적으로 그래프의 점수는 아래의 수식으로 계산한다.

   ![image](https://user-images.githubusercontent.com/33983084/111190728-b9c5d380-85fa-11eb-97a5-495d7ba4c259.png)

   sr의 결과값을 모두 더해서 c_k의 뻗어나가는 간선 개수의 절대값으로 나눈다. c_k는 c_j의 원소다. 그 후 c_k가  후보키들 중 얼마나 중요한지에 대한 점수인 PageRank점수를 곱한다. 현재 후보키가 선정된 후보키 집합에서 얼마나 어울리는지 얼마나 중요한지를 계산한다고 볼 수 있음.

   이때 감쇠계수 d를 0.85로 사용하였는데, 어떤부분을 더 가중할지 결정하는 계수이다. 서로 독립적인 점수를 내는 theme score와 sr score를 상관계수를 사용하여 서로 점수를 적대적으로 만들어준다. 본 논문에서는 0.85를 사용했기 때문에 후보키가 후보군에서 얼마나 중요한지에 대한 정보를 후보키가 주제 도메인에 맞는지보다 더 중효다고 본다.
   ![image](https://user-images.githubusercontent.com/33983084/111195603-c8fb5000-85ff-11eb-8d95-ca833a4447dc.png)

   F1@10이 2번째로 높은 모델은 Wav2Vec을 사용했음. 성능 개선을 시켰는데, 2020년에 arXiv에 올라온 딥러닝 모델에 비하면 훨씬 성능이 좋다.

   ![image](https://user-images.githubusercontent.com/33983084/111196254-6a82a180-8600-11eb-9216-3189f0d44390.png)

   위에 결과가 본 논문 실험 결과이고, 아래는 작년에 올라온 transformer기반의 generator 모델임. 특히 SemEval에서는 매우매우매우 높음.

   ![image](https://user-images.githubusercontent.com/33983084/111196090-458e2e80-8600-11eb-9014-0a343b6bdc7c.png)



**discussion...**

Exact Match로 교사학습을하면 새로운 문서에 대해서는 매우 매우 매우 못 뽑을 확률이 높음, 특히나 transformer 기반 모델일 경우 거의 똑같은 토큰의 배치가 일어날 때, 해당 keyword가 똑같이 핵심어구여야 성능이 높음... 그래서 본 논문에서는 chunking과 ner로 많은 후보군을 뽑아놓고, 주제 도메인에 완전 해당하는 키워드들을 핵심어구라고 지정하는 방식으로 접근한게 성능 개선의 키 아이디어로 추측됨.. **발상의 전환**


---
layout: "post"
title: "SQLova"
date: "2019-02-23 21:14"
category: Paper Review
tag: [NLP, wikiSQL, Natural Language to SQL, 고급]
author: KBG
---

# **SQLova**

 *"A Comprehensive Exploration on WikiSQL with Table-Aware Word Contextualization"* ([arxiv](https://arxiv.org/pdf/1902.01069.pdf))
- Natural Language to SQL task는 자연어로 된 질문을 SQL로 변경하여 DB에 질의를 통해 질문에대한 정답을 이끌어내는 task 입니다. 
- SQLova는 Natural Language to SQL task 의 대표적인 dataset인 wikiSQL의 leader board에서 기존의 모델들보다 훨씬 더 높은 성능으로 State-of-the-art 를 달성해 주목을 받은 모델입니다. (현재는 2위가 되었네요.)
- Language Representation 인 BERT와 기존연구들에 대한 사전지식이 있어야 좀 더 이해하기 수월한 논문입니다.
- 이해를 돕기 위해 wikiSQL dataset을 먼저 살펴보고 이후에 SQLova에 대해서 살펴보도록 하겠습니다.

## wikiSQL dataset

- Salesforce에서 공개한 WikiSQL dataset은 Natural Language to SQL task의 대표적인 dataset 입니다.
- wikiSQL dataset은 Relational Database의 자연어 인터페이스를 구축하기 위한 목적으로 만들어 졌습니다.

![alt text](https://github.com/BroCoLySTyLe/SQLovaReview/blob/master/images/wikiSQL1.png)

- wikiSQL dataset은 위의 그림과 같이 자연어로된 Question과 Table정보를 주고 Question의 정답을 찾기위한 **SQL**을 생성해 정답을 찾아내는 형식의 데이터셋 입니다. 


![alt text](https://github.com/BroCoLySTyLe/SQLovaReview/blob/master/images/wikiSQL2.png)

- wikiSQL dataset은 자연어 Question인 **NL** , Table의 해더정보인 **TBL** , SQL문의 ground truth인 **SQL(T)** , 자연어 Quesion의 정답인 **ANS(T)** 로 구성되어 있습니다.
  -  **SQL(**P**)** 와 **ANS(**P**)** 는 SQLova 모델의 Predicted SQL과 Predicted SQL문을 이용하여 얻은 정답 값이고 위의 데이터 예시처럼 ground truth가 잘못된 것들이 존재합니다.  
  
- wikiSQL dataset은 **SELECT문** 만을 대상으로 하고 있기 때문에 다음과 같이 **6가지의 component**를 구해내면 되는 문제로 치환됩니다.
  - **`select-column` : select-column은 테이블 헤더들(TBL) 중에서 SELECT 문에 들어갈 column을 정해줍니다.** 
    - SQL(T)에서의 Byes에 해당
  - **`select-aggregation` : select-aggragation은 위의 과정에서 정해진 SELECT column에 적용할 avg , max , min , count 등의 연산기호를 정해줍니다.** 
    - SQL(T)에서의 avg()에 해당
  - **`where-number` : WHERE 절의 조건으로 들어갈 column 갯수를 정해줍니다.** 
    - wikiSQL 에서는 WHERE절에서 and 로만 여러개의 column 조건을 이용한 데이터들만 존재합니다.
  - **`where-column` : WHERE 절의 조건으로 들어갈 column들을 정해줍니다.**
    - SQL(T)에서 AND로 연결된 Against와 Wins에 해당
  - **`where-operator` : WHERE 절에 들어갈 조건의 대소비교, 등호 등을 결정합니다.** 
    - SQL(T)에서 =와 <에 해당함 
  - **`where-value` : 자연어로된 Question(NL)로 부터 WHERE 절에 들어갈 조건의 value 값을 결정합니다.** 
    - SQL(T)에서 1076과 13에 해당

 
 ---
## **SQLova paper**
우선적으로 이 논문의 contribution을 먼저 짚고 넘어가겠습니다.
#### contribution
- Natural Language to SQL과 같은 structured data를 이용하는 task에서 BERT와 같이 문맥을 반영한 단어 정보를 이용할 수 있는 Natural Language Representation를 적용하고 그 효용성을 입증
- Table-Aware BERT 모델을 이용하여서 wikiSQL의 leader board에서 기존 state-of-the-art 성능 대비 큰 성능향상을 보인 것  

#### **Table-Aware BERT**


Table-Aware BERT는 기존의 BERT를 이용하여 자연어로 된 질의와 테이블의 헤더정보들을 효과적으로 인코딩하기 위해 만들어진 모델입니다.
기존 BERT에 대한 개념은 ([BERT paper](https://arxiv.org/pdf/1810.04805.pdf))를 참고하시거나 *TmaxAI BERT post* ([Tmax AI BERT post](https://tmaxai.github.io/post/BERT/))를 참고하시면 쉽게 이해할 것 같습니다.



![alt text](https://github.com/BroCoLySTyLe/SQLovaReview/blob/master/images/tableawareBERT.png)

위의 그림에서 보시는것 처럼 Table-Aware BERT는 하늘색으로 표시된 CLS 토큰과 빨강색으로 표시된 자연어 Question 그리고 초록색으로 표시된 테이블의 헤더들 이것들을 각각 구분하기위해 회색으로 표시된 SEP 토큰을 인풋으로 갖게 됩니다.
인풋은 기존의 BERT 처럼 워드의 임베딩값과 position embedding , segment embedding 을 더한 vector값을 인풋으로 가지게 됩니다. 
그렇게 되면 Table-Aware BERT를 통해 각각 토큰의 Hidden Vector값이 나오게 됩니다. 이 word contextualization이 반영된 Hidden Vector 값을 이용하여 뒤에 나올 3가지 model scheme을 가지고 Natural Language to SQL task의 성는을 크게 높인 것이 핵심입니다.


##### **3 model scheme**

![alt text](https://github.com/BroCoLySTyLe/SQLovaReview/blob/master/images/3model_scheme.png)

논문에서 설명하는 3가지 모델 scheme에 대해서 살펴보겠습니다. 우선 논문에서는 3가지 모델 scheme중 이 논문에서 중점적으로 다루고 있는 shallow layer에 대해서 우선 살펴보겠습니다. (SQLova는 성능이 가장 잘 나온 C scheme입니다.)

###### **Shallow Layer**

![alt text](https://github.com/BroCoLySTyLe/SQLovaReview/blob/master/images/shallow.PNG)

shallow layer는 어떠한 trainable parameter도 가지고 있지 않은 간단한 구조로 Table-Aware BERT 를 fine-tuning 하기위한 loss function으로만 구성된 layer입니다.
기존의 BERT도 contextualized language representation(BERT)의 우수성을 증명하기 위해 여러 NLP task들을 풀때 새로운 parameter 를 가지는 layer를 추가하기보다는 해당 NLP task를 풀기위한 loss function만 가지고도 좋은 성능을 낸다는 것을 보였고, 이 논문에서도 역시 Natural Language to SQL task에서의 Table-Aware BERT의 우수성을 보이기 위해 Table-Aware BERT를 fine-tuning을 하기위해 loss function으로 구성된 최소의 layer를 구성한 것이 shallow layer 입니다.


![alt text](https://github.com/BroCoLySTyLe/SQLovaReview/blob/master/images/formula1.PNG)

식 (1)을 살펴보면 SELECT 문에 들어갈 column을 정하기 위한 식 입니다. 위의 shallow-layer 그림에서 초록색에 해당하는 테이블 해더들의 히든백터값의 0번째 인덱스 값들을 softmax 하여 어떤 테이블 해더가 SELECT 문에 들어갈 column 인지 정하게 됩니다.

![alt text](https://github.com/BroCoLySTyLe/SQLovaReview/blob/master/images/formula2.PNG)

식 (2)을 살펴보면 SELECT 문에 들어갈 column에 적용할 aggregation(max, min, count, avg 등)을 정해줍니다. 식(1)에서 정해진 테이블 해더의 히든백터의 1번째 ~ 6번째 인덱스 값이 각각 aggregation(NONE ~ AVG)의 score를 가지게 되고 softmax를 통해 한가지의 aggregation을 정하게 됩니다. (NONE이 activation 될경우 aggregation 없음)


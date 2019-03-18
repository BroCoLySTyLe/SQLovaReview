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

![alt text](https://github.com/BroCoLySTyLe/SQLovaReview/blob/master/image/wikiSQL1.png)

- wikiSQL dataset은 위의 그림과 같이 자연어로된 Question과 Table정보를 주고 Question의 정답을 찾기위한 **SQL**을 생성해 정답을 찾아내는 형식의 데이터셋 입니다. 


![alt text](https://github.com/BroCoLySTyLe/SQLovaReview/blob/master/image/wikiSQL2.png)

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



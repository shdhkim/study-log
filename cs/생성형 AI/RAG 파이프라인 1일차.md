# 🧠 RAG (Retrieval-Augmented Generation) 개요

## ✅ RAG란?

- RAG는 LLM이 모르는 도메인 지식을 외부에서 검색하여 응답에 반영하는 구조
- 사용자 쿼리를 받아 관련 문서를 검색(Retrieval)하고, 그 문서로부터 생성(Generation)하는 방식

---

## ✅ Dense Vector vs Sparse Vector

| 항목              | Dense Vector                            | Sparse Vector                         |
|-------------------|------------------------------------------|----------------------------------------|
| 생성 방법         | 딥러닝 모델 (BERT, SBERT 등) 사용        | 통계 기반 (TF-IDF, BM25 등)            |
| 형태              | [0.12, -0.88, 0.42, ...] (수백 차원 실수) | 대부분 0, 소수 단어만 값 존재하는 희소 벡터 |
| 의미 파악         | 가능 (문맥 인식)                         | 어려움 (단어 일치 중심)                 |
| 검색 엔진         | 벡터 DB (FAISS, Qdrant 등)               | 전통 검색엔진 (Elasticsearch 등)         |
| 대표 용도         | 의미 기반 검색, RAG, FAQ 챗봇 등         | 키워드 검색, 뉴스/쇼핑 검색 등          |

---

## ✅ 전통적인 텍스트 검색엔진

- 단어 기반으로 문서와 쿼리를 매칭하는 검색 방식
- 주로 Sparse Vector(BM25, TF-IDF)를 사용

### 🔹 구성 요소
- **역색인 (Inverted Index)**: 단어 → 문서 맵핑
- **스코어링 알고리즘**: BM25, TF-IDF
- **Ranking**: 문서의 유사도에 따라 정렬

### 🔹 대표 기술
- **Lucene**: Java 기반 검색 라이브러리
- **Elasticsearch**: 분산 검색 시스템
- **Solr**: Lucene 기반 검색 서버
- **Whoosh**: Python 기반 간단 검색엔진

---

## ✅ 챗봇에서의 RAG 활용

> "휴가 몇 일이에요?" 같은 질문 → 내부 문서에서 검색 후 답변 생성

### 🔹 작동 흐름

```
[사용자 질문]
   ↓
[1] 쿼리 벡터화 (Dense 또는 Sparse)
   ↓
[2] 관련 문서 검색 (벡터 DB or 검색엔진)
   ↓
[3] 문서 + 질문 → LLM에 입력
   ↓
[4] LLM이 응답 생성
```

### 🔹 실시간 검색 여부
- ChatGPT + 웹브라우징 기능 → 실시간 웹 검색 가능
- 자체 RAG 시스템 → 벡터 DB에서 실시간 내부 문서 검색 가능

---

## ✅ RAG 기반 챗봇 구축 구조

1. **문서 임베딩**: 사내 문서 → Dense Vector
2. **벡터 저장소**: FAISS, Qdrant 등
3. **사용자 질문 임베딩**
4. **유사 문서 검색**
5. **LLM 응답 생성**
6. **응답 반환**

---

## ✅ 사용 기술 스택

| 구성 요소     | 기술 예시                       |
|----------------|-------------------------------|
| 벡터 임베딩     | OpenAI embedding, SBERT         |
| 벡터 DB        | FAISS, Qdrant, Pinecone         |
| 검색 엔진      | Elasticsearch, BM25             |
| LLM            | GPT-4, Claude, Mistral 등       |
| 통합 프레임워크 | LangChain, LlamaIndex, Haystack |

---

# 🔍 Advanced RAG Variants Overview 

---

## 📘 Retrieval

### ✅ Naive-RAG (최초 제안)

* **논문**: *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks* (2020)
* **기관**: Facebook AI Research (FAIR)
* **링크**: [논문 보기](https://arxiv.org/pdf/2005.11401) 
#### 🔍 설명

Naive-RAG는 최초로 등장한 RAG 모델로, 지식이 필요한 NLP 작업에 적합하도록 설계되었습니다. 기존의 단순한 문장 생성 방식이나 단순 검색 기반 질의응답 시스템의 한계를 극복하기 위해 도입되었습니다.

* **Retriever**: 질문과 관련된 정보를 찾는 역할. 이때 **Dense Passage Retrieval (DPR)** 방식을 사용하여, 문장을 임베딩한 뒤 벡터 유사도를 기준으로 문서를 검색합니다.
* **Generator**: 검색된 문서를 기반으로 텍스트를 생성하는 역할. 대표적으로 **BART**와 같은 시퀀스-투-시퀀스(seq2seq) 모델이 사용됩니다.
* **End-to-End 학습 가능**: 검색기와 생성기를 함께 학습시킬 수 있어, 검색된 문서가 생성에 어떻게 활용되는지를 모델이 스스로 최적화할 수 있습니다.

#### 📌 핵심 기여

* 검색 + 생성 통합 아키텍처 제안
* 기존 생성 모델이 지니지 못한 **외부 지식 반영 능력** 부여
* 학습 없이도 **새로운 지식 문서만 추가해 업데이트 가능**

#### 📚 연구적 의의

* 모델 안의 지식과 외부 문서를 결합한 첫 시도
* 기존의 단일 생성기보다 훨씬 풍부한 정보 기반 응답 생성 가능

---

### ✅ Advanced-RAG (고도화 구조)

* **논문**: *Retrieval-Augmented Generation for Large Language Models: A Survey* (2023.12)
* **기관**: Tongji University
* **링크**: [논문 보기](https://arxiv.org/pdf/2312.10997)
#### 🔍 설명

Advanced-RAG는 Naive-RAG보다 한 단계 발전된 구조로, 전체 RAG 시스템을 **검색(Retrieval) → 증강(Augmentation) → 생성(Generation)** 세 단계로 나누고 각 요소를 고도화합니다.

* **다중 검색기(Multi-Retriever)** 도입: 다양한 방식의 검색기를 조합해 검색 성능 향상
* **Reranking**: 검색된 문서를 다시 평가하여 더 정밀한 순위로 정렬
* **Augmentation**: 문서를 Chunking, Filtering, Fusion 등으로 가공하여 생성기에게 최적의 입력 제공
* **Generation**: 지시 기반 생성(Instruct-tuned generation), Self-consistency, Self-critique 등을 도입해 더 정확한 응답 생성

#### 📌 핵심 기여

* 단순 Top-k 검색을 넘어서 질의에 최적화된 정제된 정보 제공
* 문서 구성과 생성 품질 모두를 개선함으로써 **전반적인 응답 정확도 향상**
* 모든 구성 요소를 모듈화하여 연구자 및 개발자가 유연하게 활용 가능

#### 📚 연구적 의의

* 기존 RAG의 단순 파이프라인을 넘어서 **전략 기반 검색 구조로 발전**
* 검색 품질과 생성 품질을 동시에 높일 수 있는 통합 설계 지향

---

## 💬 Query-Aware RAG

### ✅ Self-RAG (자기 반성 기반 RAG)

* **논문**: *Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection* (2023.10)
* **기관**: University of Washington, AI2, Amazon AWS AI
* **링크**: [논문 보기](https://arxiv.org/pdf/2310.11511)
#### 🔍 설명

Self-RAG는 RAG 시스템의 고질적인 문제인 **hallucination**(사실과 다른 내용을 생성하는 오류)을 줄이기 위해 **자기 반성(self-reflection)** 개념을 도입합니다.

* **초기 생성**: 기존 RAG처럼 1차 응답을 생성
* **자기 비판(Critique)**: 생성된 응답이 실제 질문에 얼마나 충실한지 모델 스스로 평가
* **재검색(Retrieval)**: 부족한 정보가 감지되면, 새롭게 문서를 검색함
* **응답 개선(Revision)**: 보완된 문서를 바탕으로 더 신뢰성 있는 응답 생성

#### 📌 핵심 기여

* "답변 → 점검 → 재검색 → 개선"의 **루프 구조 도입**
* Self-Critique 기능으로 **모델 자체 fact-checking 가능**
* 별도 정답 라벨 없이도 **자기 지도 학습(self-supervised learning)** 수행 가능

#### 📚 연구적 의의

* 모델이 스스로 자신의 응답 품질을 검토하고 수정할 수 있는 능력 부여
* 단순한 생성 모델에서 **스스로 판단하는 AI Agent 구조로 확장**

---

### ✅ Corrective-RAG (검색 품질 보정 RAG)

* **논문**: *Corrective Retrieval-Augmented Generation* (2024.01)
* **기관**: Google DeepMind
* **링크**: [논문 보기](https://arxiv.org/pdf/2401.15884)
#### 🔍 설명

Corrective-RAG는 검색된 문서 자체가 **부정확하거나 부족할 수 있다**는 현실적인 문제를 해결하기 위해 고안된 프레임워크입니다.

* **Retrieval Evaluator**: 검색된 문서들의 품질을 평가하여 **신뢰도 점수**를 부여
* **웹 검색 통합**: 제한된 문서 DB 대신 **외부 웹 검색도 활용**하여 coverage 보완
* **Decompose-then-Recompose Algorithm**: 중요한 정보만 추출 → 필터링 → 재조합하여 생성기에 제공

#### 📌 핵심 기여

* 검색 정확도를 **스스로 평가하고 보정**하는 메커니즘 도입
* 검색 결과가 부정확하더라도 **후처리로 정보 품질 향상 가능**
* 기존 RAG 시스템에 쉽게 통합 가능한 **모듈형 구조**

#### 📚 연구적 의의

* 검색 품질의 취약점을 효과적으로 보완할 수 있는 실용적 확장
* RAG 시스템의 **신뢰성과 견고성 향상**

---

### ✅ Adaptive-RAG (질문 복잡도 적응형)

* **논문**: *Adaptive-RAG: Learning to Adapt Retrieval-Augmented Large Language Models through Question Complexity* (2024.03)
* **기관**: KAIST
* **링크**: [논문 보기](https://arxiv.org/pdf/2403.14403)
#### 🔍 설명

Adaptive-RAG는 질문의 난이도(복잡도)에 따라 검색 전략을 **자동 조절**하는 프레임워크입니다. 모든 질문에 동일한 RAG 절차를 적용하는 기존 방식에서 벗어나 다음과 같이 구분합니다:

1. **간단한 질문** → 검색 없이 내부 지식으로만 응답
2. **보통 수준 질문** → 1회 검색 후 응답 생성
3. **복잡한 질문** → 여러 번 검색 후 정보 종합하여 응답 생성

* 질의 복잡도는 자동 학습된 **Query Complexity Classifier**로 판단됩니다.

#### 📌 핵심 기여

* 질의 난이도에 따른 **동적 전략 선택**
* **수작업 라벨링 없이도** 분류기 학습 가능
* 계산 리소스를 절약하면서 **응답 품질 유지**

#### 📚 연구적 의의

* 효율성 + 정확성 동시 확보 가능
* 다양한 난이도의 질문을 유연하게 대응할 수 있는 **지능형 검색 시스템 기반 제시**

---

# 📘 RAG 구조적 진화: 최신 RAG 프레임워크 세 가지

## 1. Graph RAG – 문서 집합을 지식 그래프로 구조화

- **논문명**: From Local to Global: A Graph RAG Approach to Query-Focused Summarization  
- **기관**: Microsoft (2024.04)  
- **링크**: [논문 보기](https://arxiv.org/pdf/2404.16130)

### 📌 주요 개념

기존의 Naive RAG는 단순 임베딩 검색 기반이라서 "이 문서 전체에서 주제는 무엇인가?"와 같은 **전반적인 질문 처리에 한계**가 있었음. 이에 따라 Graph RAG는 문서 간의 **엔티티-관계 구조를 그래프로 표현**하고, 이를 바탕으로 요약 및 질의응답을 수행.

### 🔧 핵심 구성

- **지식 그래프 생성**: LLM을 활용하여 문서에서 엔티티(개체)와 관계를 추출하여 그래프 구조로 표현
- **커뮤니티 요약**: 밀접하게 연결된 노드 군집을 ‘커뮤니티’로 식별하고, 각 커뮤니티 단위로 요약문을 생성
- **질문 응답**: 사용자의 질문에 대해 관련된 커뮤니티의 요약을 활용해 부분 응답을 생성하고, 이를 종합하여 최종 응답 생성

### 🧠 연구적 의의

- 기존 청크 기반 검색에서 → **구조화된 지식 기반 Retrieval 방식**으로 전환
- LLM을 단순 생성기가 아니라 **요약기 + 생성기** 역할로 확장
- 엔터프라이즈 레벨에서 유용한 고차원 요약/응답 시스템 구현 가능성 제시

---

## 2. Modular RAG – 레고처럼 재구성 가능한 프레임워크

- **논문명**: Modular RAG: Transforming RAG Systems into LEGO-like Reconfigurable Frameworks  
- **기관**: Shanghai Jiao Tong University, East China Normal University (2024.07)  
- **링크**: [논문 보기](https://arxiv.org/pdf/2407.21059v1)

### 📌 주요 개념

RAG 시스템을 레고 블럭처럼 재구성 가능하게 만들자는 접근. **모듈, 하위 모듈, 연산자**로 구분해 복잡한 시스템을 유연하게 설계.

### 🔧 핵심 구성

1. **모듈 (Module)**: 검색, 인덱싱, 생성 등 RAG의 핵심 단계를 담당  
2. **하위 모듈 (Sub-modules)**: 각 모듈 내의 세부 기능 (e.g., 쿼리 확장, 검색기 교체 등)
3. **연산자 (Operators)**: 실제 연산 수행 (e.g., 슬라이딩 윈도우, 메타데이터 삽입)

### 🧠 연구적 의의

- **모듈화 설계**로 유지보수 및 교체 용이성 확보
- **유연한 시스템 구성**으로 다양한 시나리오 대응 가능
- **새 기술 도입과 성능 개선**을 손쉽게 수행

---

## 3. Agentic RAG – 에이전트 기반으로 도구 활용 확장

- **논문명**: Agentic Retrieval-Augmented Generation: A Survey on Agentic RAG  
- **기관**: Cleveland State University, Roux Institute at Northeastern University (2025.01)  
- **링크**: [논문 보기](https://arxiv.org/pdf/2501.09136)

### 📌 주요 개념

기존 RAG 시스템을 확장하여, **에이전트 기반**으로 작동하는 구조. 단순 질의응답을 넘어서 **계획, 도구 활용, 반성(reflection), 다중 에이전트 협업**이 가능한 시스템.

### 🔧 핵심 구성

- **동적 검색 전략**: 에이전트가 실시간으로 검색 전략을 수정하여 문맥에 최적화
- **문맥 이해 개선**: 반복적 피드백 루프를 통해 문맥에 대한 이해도를 지속적으로 향상
- **다중 에이전트 협업**: 역할 분담과 협업을 통해 복잡한 작업도 수행 가능

### 🧠 연구적 의의

- **자율성과 적응성**을 갖춘 RAG 시스템 설계 가능
- **에이전트 패턴 활용**으로 복잡한 업무 자동화 가능 (예: 계획 수립, 반성, 도구 활용)
- 의료, 금융 등 산업 분야에의 적용 가능성 시사
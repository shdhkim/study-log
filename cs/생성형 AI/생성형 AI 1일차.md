# LLM (Large Language Model)

**LLM**(Large Language Model)은 대규모 데이터를 학습한 인공지능 언어 모델로,  
자연어 처리(NLP) 작업을 수행하는 딥러닝 모델임. GPT, BERT, LLaMA 같은 모델들이 대표적인 LLM에 속함.

## 대규모 학습 데이터
인터넷, 책, 논문, 코드 등 다양한 텍스트 데이터를 학습하여 방대한 지식을 보유.

## 사전 학습(Pre-training) + 미세 조정(Fine-tuning)
기본적으로 거대한 양의 텍스트로 **사전 학습(Pre-training)** 한 후,  
특정 작업(예: 코딩, 의료, 법률)에 맞춰 **미세 조정(Fine-tuning)** 진행.

## 자연어 이해 및 생성
사람과 유사한 수준의 텍스트를 생성하고, 문맥을 이해하여 대화를 수행할 수 있음.

## 문맥 유지 및 추론 가능
몇 문장 이상의 컨텍스트를 기억하고, 논리적으로 이어지는 답변 생성 가능.

## 어텐션(Attention)
특정 입력(예: 문장)에서 중요한 부분에 집중하는 기법.

- 주로 **RNN 기반의 인코더-디코더 모델**에서 사용됨.
- 입력(소스)과 출력(타겟)이 다를 때 적용됨.

### 어텐션이 필요한 이유?
기존 **Seq2Seq(인코더-디코더)** 모델은 RNN을 사용했는데,  
긴 문장을 처리할 때 정보가 손실됨 (**장기 의존성 문제**).  
매 시점에 전체 문맥을 고려하지 못함.

## 셀프 어텐션(Self-Attention)
입력 문장 내에서 단어 간의 관계를 학습하는 기법.

- 입력(소스)과 출력(타겟)이 같을 때 사용됨.
- **Transformer 모델(GPT, BERT 등)**의 핵심 기술.
- 한 문장 안에서 단어 간의 연관성을 학습하여 문맥을 이해함.

### 셀프 어텐션이 중요한 이유?
- 문장 내에서 **멀리 떨어진 단어들 간의 관계도 고려 가능**.
- 병렬 연산이 가능해서 **연산 속도가 RNN보다 훨씬 빠름**.

---

## 토큰(Token)
**생성형 AI(LLM)에서 토큰(Token)은 텍스트를 처리하고 모델이 이해하는 최소 단위**.

예제: `"I love AI models!"`

1. **공백 기준 단어 단위 토큰화 (Word Tokenization)**  
   `["I", "love", "AI", "models!"]`
2. **서브워드 토큰화 (Subword Tokenization - BPE, WordPiece)**  
   `["I", "love", "AI", "model", "s", "!"]`
3. **문자 단위 토큰화 (Character Tokenization)**  
   `["I", " ", "l", "o", "v", "e", " ", "A", "I", " ", "m", "o", "d", "e", "l", "s", "!"]`

---

## 할루시네이션(Hallucination)
**생성형 AI(LLM)가 사실이 아닌 정보를 생성하는 현상**을 의미함.  
AI가 존재하지 않는 데이터나 잘못된 내용을 진짜처럼 만들어내는 문제.

### 원인:
- 모델이 **정확한 사실을 기억하는 것이 아니라**, 확률적으로 가장 적절한 단어를 예측하는 방식으로 동작함.
- **모델이 학습 데이터에 없는 정보일 때** → 그럴듯한 내용을 생성함.
- **컨텍스트가 부족할 때** → 엉뚱한 답을 생성함.
- **긴 문맥을 처리하면서 정보가 왜곡될 때** → 기존 내용과 다르게 답변함.

### 할루시네이션 유형:
| 유형              | 예시 |
|------------------|-----------------------------------------------------------|
| **사실 왜곡** (Factually Incorrect) | "대한민국 대통령은 이순신이다." |
| **출처 조작** (Fabricated Citations) | "논문 'AI 혁명'은 Nature에 실렸다." (존재하지 않음) |
| **비논리적 연결** (Incoherent Responses) | "고양이는 알레르기를 치료하는 약이다." |
| **과장된 정보** (Overgeneralization) | "모든 프로그래머는 하루에 10시간씩 코딩한다." |

---

## LLM의 동작 방식:
1. 사용자의 입력(프롬프트)을 **토큰화**
2. 입력된 문맥을 바탕으로 **다음 단어를 확률적으로 예측**
3. 가장 높은 확률의 단어를 선택하여 문장을 **생성**
4. 반복하여 문장을 확장

---

## Transformer의 기본 구조
**Transformer는 인코더-디코더(Encoder-Decoder) 구조**로 되어 있음.

### 인코더(Encoder)
- 입력 문장을 받아서 **어텐션을 통해 문맥을 이해**하고, 이를 압축된 벡터로 변환.
- 여러 개의 **Self-Attention Layer**와 **Feed Forward Layer**로 구성됨.

### 디코더(Decoder)
- **인코더가 생성한 정보를 바탕으로 출력을 생성**.
- 문장을 한 번에 생성하는 것이 아니라, **토큰을 하나씩 예측하면서 생성**.
- 다음 단어를 예측할 때 **마스크드 어텐션(Masked Attention)**을 사용하여 미래 단어를 보지 않도록 함.

---

## 첨크(Chunk)
긴 문서를 작은 단위로 나누는 기법. **LLM, 검색, RAG에서 필수적인 기법**.  
토큰 기반, 문장 기반, 단락 기반 등 다양한 첨크 방법이 존재함.

---

## RAG (Retrieval-Augmented Generation)
**LLM이 외부 데이터베이스에서 검색(Retrieval)한 정보를 바탕으로 응답을 생성(Generation)하는 방식**.  
기존 LLM이 **사전 학습된 데이터로만 답변**하는 것과 달리, 최신 정보를 반영할 수 있음.

---

## Word Embedding
단어를 **고정된 크기의 밀집 벡터(Dense Vector)** 로 표현하는 방법.  
의미가 비슷한 단어들은 벡터 공간에서 가까운 위치에 있도록 훈련됨.  
**코사인 유사도(Cosine Similarity)** 등을 사용하여 유사도를 측정 가능.

---

## 소프트맥스(Softmax) 함수
- 여러 개의 실수 값을 **확률 분포로 변환하는 함수**.
- **입력된 값들을 0과 1 사이의 확률 값**으로 변환하고, 전체 합이 1이 되도록 정규화.
- 주로 **다중 분류(Multi-Class Classification)** 문제에서 사용됨.

---

## BERT의 학습 과정
1. **사전 학습(Pretraining)**: 대규모 데이터에서 일반적인 언어 패턴 학습.
2. **파인튜닝(Fine-Tuning)**: 특정 태스크(감성 분석, 질의응답 등)에 맞게 미세 조정.

---

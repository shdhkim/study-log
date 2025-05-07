# 📘 벡터 DB를 활용한 비정형 텍스트 의미 검색

## ✅ 개요

벡터 DB는 임베딩된 벡터 데이터를 저장하고, 유사도 기반으로 검색할 수 있도록 설계된 데이터베이스입니다.  
비정형 데이터(텍스트, 이미지 등)를 임베딩 모델로 벡터화한 후, 유사도 기반 검색이 가능합니다.

---

## ✅ 전체 흐름 요약

```
[문서 텍스트들] ──▶ [임베딩 모델] ──▶ [벡터] ──▶ [벡터 DB에 저장]

[질문 쿼리] ──▶ [임베딩 모델] ──▶ [질문 벡터] ──▶ [벡터 DB에서 유사 문서 검색]
```

---

## ✅ 사용 예제 (ChromaDB + SentenceTransformer)

```python
import chromadb
from sentence_transformers import SentenceTransformer

# 1. 클라이언트 및 컬렉션 생성
client = chromadb.Client()
collection = client.create_collection(name="docs")

# 2. 문장 벡터화
model = SentenceTransformer('all-MiniLM-L6-v2')
texts = [
    "고양이는 애교가 많고 독립적인 동물입니다.",
    "인공지능은 인간처럼 학습하고 추론할 수 있는 기술입니다.",
    "강아지는 주인을 잘 따르고 활동적입니다.",
    "머신러닝은 인공지능의 한 분야로, 데이터를 이용해 모델을 학습시킵니다."
]
embeddings = model.encode(texts).tolist()

# 3. 벡터 DB에 문서 저장
collection.add(
    documents=texts,
    embeddings=embeddings,
    ids=["doc1", "doc2", "doc3", "doc4"]
)

# 4. 사용자 질문 벡터화 + 검색
query = "기계학습이란 뭘까?"
query_vector = model.encode([query]).tolist()[0]
results = collection.query(query_embeddings=[query_vector], n_results=2)

# 5. 결과 출력
for doc in results['documents'][0]:
    print("-", doc)
```

---

## ✅ 핵심 구조

| 단계 | 설명 |
|------|------|
| 문서 저장 | 문장을 벡터로 변환하고 DB에 저장 |
| 질문 쿼리 | 질문도 벡터로 변환 |
| 검색 수행 | 질문 벡터와 저장된 벡터들 간 유사도 계산 후 Top-K 반환 |

---

## ✅ 자주 사용하는 유사도 지표

- **Cosine Similarity (코사인 유사도)** → 방향이 비슷한 벡터 찾기
- **Euclidean Distance (L2 거리)** → 거리 기반으로 가까운 벡터 찾기

---

## ✅ 활용 사례

- 🔍 의미 기반 문서 검색 (Semantic Search)
- 💬 FAQ 자동 응답
- 📚 RAG (Retrieval-Augmented Generation)
- 🧠 유사 문장 추천

---


# 📦 ChromaDB Collection 구성 및 활용 가이드

## ✅ Collection = RDB의 테이블과 같은 역할
- 벡터 DB에서 `Collection`은 데이터를 저장하는 단위이며, RDB로 치면 테이블, 엑셀로 치면 시트 개념입니다.
- 하나의 Collection에는 벡터, 메타데이터, 문서 등의 정보가 함께 저장됩니다.

---

## ✅ Collection 구성요소

| 항목 | 설명 |
|------|------|
| **벡터 (embedding)** | 문서 또는 데이터의 임베딩 벡터값 |
| **ID** | 각 벡터의 고유 식별자 |
| **Metadata** | 벡터에 연결된 JSON 형식의 추가 정보 |
| **Document** | 원본 문서 내용 |
| **URI** | 외부 리소스 경로 (이미지, 오디오 등) |

---

## ✅ 컬렉션 관리

### 📌 리스트 및 카운트
```python
client.count_collections()   # Collection 개수 조회
client.list_collections()    # Collection 목록 조회
```

### 📌 컬렉션 삭제
```python
client.delete_collection(name="height_weight")
```

### 📌 컬렉션 생성 or 존재 시 조회
```python
client.get_or_create_collection(
    name="chroma_tutorial",
    metadata={"hnsw:space": "cosine"}
)
```

---

## ✅ 데이터 추가: `collection.add()`

```python
collection.add(
    ids=id_list,                     # 유일한 식별자
    embeddings=embedding_list,       # 벡터 리스트
    metadatas=metadata_list,         # 메타데이터 dict
    documents=doc_list,              # 원문 문서
    uris=uri_list                    # 외부 URI (옵션)
)
```

---

## ✅ 데이터 조회: `collection.get()`

```python
collection.get(
    offset=0,
    limit=3,
    where={"키": {"$gte": 170}},  # 메타데이터 조건
    where_document={"$contains": "name"}  # 문서 조건
)
```

### ▶ Where 조건
- `$eq`, `$ne`, `$gt`, `$gte`, `$lt`, `$lte`
- `$and`, `$or`

### ▶ Where Document 조건
- `$contains`, `$not_contains`

---

## ✅ 유사도 기반 검색: `collection.query()`

```python
collection.query(
    query_embeddings=[[183, 78]],
    where={"키": {"$gte": 170}},           # 메타데이터 조건
    where_document=None,                   # 문서 검색 조건
    n_results=11,
    include=["metadatas", "embeddings"]
)
```

---

## ✅ 데이터 수정: `collection.update()`

```python
collection.update(
    ids=["손흥민"],                         # 수정할 ID
    embeddings=None,                       # 수정할 임베딩 (없으면 유지)
    metadatas=[{"키": 183, "몸무게": 78, "나이": 32}],
    documents=None
)
```

---

## ✅ 데이터 삭제: `collection.delete()`

```python
collection.delete(
    ids=None,
    where={"키": {"$lte": 170}},              # 메타데이터 조건
    where_document={"$contains": "name"}      # 문서 조건
)
```

---

**📌 Tip:** ChromaDB는 `컬렉션 단위로 벡터+문서+메타데이터`를 묶어서 다루는 것이 핵심입니다. 벡터 유사도 검색 외에도 조건 기반 필터링이 가능하므로 RDB와 벡터 검색의 장점을 모두 결합할 수 있습니다.

---

# 👓 피부톤 기반 안경 색상 추천 알고리즘

## ✅ 개요
이 알고리즘은 사용자의 **피부톤**을 기반으로 **어울리는 안경 색상**을 추천합니다.  
색상 비교에는 HSL 또는 CIE Lab 색공간을 사용하며, **유사한 톤 매칭** 또는 **보색 대비 매칭** 전략을 적용할 수 있습니다.

---

## ✅ 주요 개념

| 요소 | 설명 |
|------|------|
| 피부톤 | RGB 또는 HEX → Lab/HSL 변환 필요 |
| 안경 색상 | 후보 색상들을 미리 Lab/HSL로 준비 |
| 조화 전략 | 유사색 매칭, 보색 대비 등 |
| 색상 유사도 | ΔE (Lab) 또는 Hue/Saturation 거리 (HSL) |

---

## ✅ 알고리즘 흐름 

```python
1. 사용자 얼굴 사진에서 피부 RGB 색상 추출
2. 피부색을 Lab 또는 HSL로 변환
3. 안경 색상 후보들을 Lab 또는 HSL로 변환
4. 조화 전략 선택:
    - 유사 톤 (ΔE 또는 Hue 거리 최소화)
    - 대비 톤 (명도/채도 차이 극대화)
5. 조건을 만족하는 Top-K 안경 색상 추천
```

---

## ✅ 예시 전략: Lab 기반 유사 색상 매칭

```python
from colormath.color_objects import sRGBColor, LabColor
from colormath.color_conversions import convert_color
from colormath.color_diff import delta_e_cie2000

def rgb_to_lab(rgb):
    return convert_color(sRGBColor(*rgb, is_upscaled=True), LabColor)

def recommend_glasses(skin_rgb, glasses_color_dict, top_k=3):
    skin_lab = rgb_to_lab(skin_rgb)
    scores = []
    for name, rgb in glasses_color_dict.items():
        g_lab = rgb_to_lab(rgb)
        delta = delta_e_cie2000(skin_lab, g_lab)
        scores.append((name, delta))
    scores.sort(key=lambda x: x[1])
    return scores[:top_k]
```

---

## ✅ 전략별 응용

| 전략 | 설명 |
|------|------|
| 유사 색 조합 | 피부톤과 유사한 안경색 → 부드럽고 자연스러운 인상 |
| 대비 색 조합 | 피부톤과 명확히 구분되는 색 → 또렷하고 세련된 인상 |
| 톤온톤 | 명도/채도 유사한 계열로 연결 → 감각적인 연출 가능 |

---

## ✅ 예시 결과

- 피부 RGB: (230, 200, 170) → 웜톤
- 추천 색상: 로즈골드, 샴페인, 라이트 브라운 (ΔE 기준 최저)

---

## ✅ 요약

- RGB → Lab/HSL 변환이 핵심
- Lab은 ΔE로 유사도 정량화 가능
- 조화 전략(유사/보색)에 따라 결과 달라짐
- 사용자별 맞춤형 안경 추천이 가능해짐!

---

# 🎨 벡터db를 활용한 안경 색상 추천 

## ✅ 목표
사용자의 얼굴 이미지에서 **피부톤을 추출**하고,  
이를 벡터로 변환하여 **안경 색상과 유사도 기반으로 추천**합니다.

---

## 1️⃣ 피부톤 벡터 추출

### 🔹 입력
- 사용자 얼굴 이미지 (예: 이마, 볼 등 피부가 드러난 부분)

### 🔹 처리 절차
1. 이미지에서 ROI(Region of Interest) 추출
2. 해당 부분의 RGB 평균 계산
3. RGB → Lab 색공간으로 변환 (사람의 시각과 유사)

```python
from colormath.color_objects import sRGBColor, LabColor
from colormath.color_conversions import convert_color

skin_rgb = (230, 200, 170)
skin_lab = convert_color(sRGBColor(*skin_rgb, is_upscaled=True), LabColor)
skin_vector = [skin_lab.lab_l, skin_lab.lab_a, skin_lab.lab_b]
```

---

## 2️⃣ 안경 색상 벡터 사전 구축 (벡터 DB 저장)

### 🔹 예시 안경 색상 벡터 (Lab)

```python
glasses_colors = {
  "rose_gold": [70.2, 14.1, 20.0],
  "silver": [80.5, 0.5, 0.1],
  "navy": [30.0, 3.0, -40.0]
}
```

### 🔹 벡터 DB 저장 구조

| ID        | Embedding (Lab 벡터)     | Metadata          |
|-----------|---------------------------|-------------------|
| rose_gold | [70.2, 14.1, 20.0]        | {"color": "rose"} |
| silver    | [80.5, 0.5, 0.1]          | {"color": "silver"} |
| navy      | [30.0, 3.0, -40.0]        | {"color": "blue"}  |

```python
collection.add(
    ids=["rose", "silver", "navy"],
    embeddings=[...],
    metadatas=[...]
)
```

---

## 3️⃣ 벡터 DB 유사도 검색 수행

```python
results = collection.query(
    query_embeddings=[skin_vector],  # [L, a, b]
    n_results=3,
    include=["metadatas", "distances"]
)
```

---

## 4️⃣ 유사도 계산 방식 (내부적으로)

- 기본: **L2 거리** 또는 **Cosine 유사도**
```text
distance = sqrt( (L1 - L2)^2 + (a1 - a2)^2 + (b1 - b2)^2 )
```

- 거리(ΔE 또는 L2)가 **작을수록 유사한 색상**

---

## 5️⃣ 전략별 추천 방식

| 전략 | 비교 기준 | 추천 방식 |
|------|------------|-----------|
| 유사 색상 | ΔE (Lab 거리) | 거리 최소화 |
| 대비 색상 | | 밝기(L) 또는 채도(a, b) 차이 최대화 |

---

## ✅ 전체 흐름 요약

```
[사용자 이미지]
    └─▶ 피부 RGB 추출
        └─▶ Lab 변환 (벡터화)
            └─▶ 안경 벡터 DB 유사도 검색
                └─▶ Top-K 안경 추천 결과 반환
```

---

## ✅ 결과 예시

- 피부톤: [72.3, 9.1, 18.6]
- 추천 색상 (유사): 로즈골드, 베이지
- 추천 색상 (대비): 네이비, 블랙

---

## ✅ 장점

- 사람 눈의 인지 기반 Lab 색공간 사용
- 빠른 벡터 유사도 계산 (Top-K 추천)
- 룰 기반보다 더 **개인화된 추천 가능**

---


# 🧠 벡터 청킹과 중첩 전략 

## ✅ 개요

벡터 청킹은 긴 텍스트를 나눠 벡터 DB에 임베딩할 수 있도록 **작은 단위로 나누는 과정**입니다.  
청킹을 어떻게 하느냐에 따라 **검색 정확도, 문맥 유지력, LLM 응답 품질**이 크게 달라집니다.

---

## 1️⃣ 왜 벡터 청킹이 필요한가?

| 이유 | 설명 |
|------|------|
| LLM 입력 길이 제한 | 한 번에 처리할 수 있는 텍스트 크기 제한 (ex. 4k~32k tokens) |
| 긴 문서 = 정보 뭉개짐 | 전체 임베딩 시 중요한 정보가 희석됨 |
| 정밀 검색 | 특정 문장/주제를 검색하려면 세밀한 청킹 필요 |

---

## 2️⃣ 청킹 단위 종류

| 청킹 방식 | 설명 | 장점 | 단점 |
|-----------|------|------|------|
| 문장 단위 | 문장마다 나눔 | 경계 명확, 의미 단절 없음 | 문맥 짧음 |
| 문단 단위 | 단락마다 나눔 | 흐름 좋음 | 길이 불균형 가능 |
| 고정 문자 수 | 500자 등 | 구현 간단 | 의미 단절 위험 |
| 토큰 기반 | 512토큰, 중첩 가능 | LLM 최적화 | 중간 단어 끊김 가능 |

---

## 3️⃣ 중복(Overlap)의 역할

중첩은 이전 청크의 일부 내용을 다음 청크에 포함시켜 **문맥을 이어주는 방법**입니다.

### 예시

```
Chunk 1: 문장1. 문장2. 문장3.
Chunk 2: 문장3. 문장4. 문장5.   ← 문장3이 겹침
```

| 효과 | 설명 |
|------|------|
| 문맥 연결성 ↑ | 앞뒤 문장의 단절 방지 |
| 검색 recall ↑ | 하나의 정보가 여러 청크에 포함되어 더 잘 검색됨 |
| LLM 이해도 ↑ | 청크 내 문맥이 자연스러움 |

---

## 4️⃣ 중첩이 오히려 해가 되는 경우

| 상황 | 설명 |
|------|------|
| 중복 문장 너무 많음 | 검색 결과 중복 (redundancy) 발생 |
| 의미 단절 지점에 중첩됨 | 문장 파편화로 오히려 혼란 유발 |
| LLM 입력 길이 초과 | 불필요한 중복으로 토큰 낭비 |

---

## 5️⃣ 중복 없이, 여러 문장 청킹 전략 

> ✅ 중복 없이 → 응답 다양성 ↑  
> ✅ 여러 문장 묶기 → 문맥 유지 OK

```python
# 문장 3개씩 묶는 청킹 예시
[
  "문장1. 문장2. 문장3.",
  "문장4. 문장5. 문장6."
]
```

- 중복 없음 → DB 용량 효율
- 문맥 단위 유지 → 의미 단절 방지
- 검색 결과 다양성 증가

---

## 6️⃣ 청크 간 연결성이란?

> "청크 간 연결성"이란 청크 A에서 이야기한 내용이 청크 B에서도 **자연스럽게 이어지는 정도**입니다.

### 예시 (연결성 낮음)

```
Chunk 1: 피카츄는 귀엽다.
Chunk 2: 진화하면 라이츄가 된다.
```
→ "피카츄가 진화"인지 알 수 없음

### 예시 (연결성 높음)

```
Chunk: 피카츄는 귀엽다. 진화하면 라이츄가 된다.
```

| 연결성 효과 | 설명 |
|-------------|------|
| 문맥 단절 방지 | 추론 정확도 ↑ |
| 질문 대응력 향상 | 앞뒤 단서가 같은 청크에 있을 확률 증가 |
| LLM 오해 방지 | 연결된 흐름을 따라갈 수 있음 |

---

## 7️⃣ 정리: 언제 어떤 전략?

| 전략 | 사용 추천 상황 |
|------|----------------|
| 문장 단위 + 중복 없음 | 기본값. 빠르고 깔끔한 QA/RAG |
| 문장 단위 + 중첩 | 문맥 중요하거나, recall 향상이 중요한 경우 |
| 문단 단위 청킹 | 설명형 문서, 보고서 등 자연스러운 구조 |
| 토큰 기반 + 중첩 | LangChain, LLM 기반 RAG 시스템에 적합 |

---

## ✅ 요약

- 청킹은 **검색 정확도와 LLM 응답 품질**을 결정짓는 핵심
- 중첩은 문맥 연결과 recall을 도와주지만, **무조건 좋은 것은 아님**
- 중복 없이 여러 문장을 묶는 전략이 실용적이며 추천됨
- 청크 간 연결성은 LLM의 이해도를 크게 좌우함

---




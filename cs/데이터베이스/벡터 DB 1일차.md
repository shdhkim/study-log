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

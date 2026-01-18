
# MyBatis ResultMap · 동적 SQL 종합 정리 

본 문서는 MyBatis 핵심 개념을 하나의 Markdown 파일로 정리한 자료다.
ResultMap의 객체 동일성 문제를 **부모 <id> 누락 / 자식 <id> 누락** 두 가지 케이스로 모두 설명한다.

---

## 1. ResultMap의 본질

- SQL ResultSet → 객체 / Map 변환
- 객체 동일성(identity) 관리
- JOIN 결과를 1:N 구조로 조립

---

## 2. `<id>`의 진짜 의미

- PK 선언이 아님
- MyBatis 내부 객체 캐시의 Key
- 같은 `<id>` → 같은 객체 재사용

---

## 3. 부모 `<id>` 누락 케이스 (객체 분리)

### 3.1 실제 테이블

**ORDERS**

| order_id |
|---------|
| 1 |

**ORDER_ITEM**

| item_id | order_id |
|--------|---------|
| 1 | 1 |
| 2 | 1 |
| 3 | 1 |

### 3.2 JOIN 결과

| ROW | order_id | item_id |
|----:|---------:|--------:|
| 1 | 1 | 1 |
| 2 | 1 | 2 |
| 3 | 1 | 3 |

### 3.3 잘못된 resultMap (부모 `<id>` 없음)

```xml
<resultMap id="orderMap" type="Order">
  <result property="orderId" column="order_id"/>
  <collection property="items" ofType="Item">
    <id property="itemId" column="item_id"/>
  </collection>
</resultMap>
```

### 3.4 결과

```
Order(1)
 └─ Item(1)

Order(1)
 └─ Item(2)

Order(1)
 └─ Item(3)
```

원인
- Order 객체를 식별할 기준이 없음
- row마다 새 Order 생성

---

## 4. 자식 `<id>` 누락 케이스 (중복 자식)

### 4.1 실제 테이블

**ORDERS**

| order_id |
|---------|
| 1 |

**ORDER_ITEM**

| item_id | order_id | item_name |
|--------|---------|-----------|
| 1 | 1 | 키보드 |
| 1 | 1 | 키보드 |
| 1 | 1 | 키보드 |

(조인 / 뷰 / 이력 테이블로 인해 같은 item이 여러 row로 나오는 상황)

---

### 4.2 JOIN 결과

| ROW | order_id | item_id | item_name |
|----:|---------:|--------:|-----------|
| 1 | 1 | 1 | 키보드 |
| 2 | 1 | 1 | 키보드 |
| 3 | 1 | 1 | 키보드 |

---

### 4.3 잘못된 resultMap (자식 `<id>` 없음)

```xml
<resultMap id="orderMap" type="Order">
  <id property="orderId" column="order_id"/>

  <collection property="items" ofType="Item">
    <result property="itemId" column="item_id"/>
    <result property="itemName" column="item_name"/>
  </collection>
</resultMap>
```

---

### 4.4 결과

```
Order(1)
 ├─ Item(1)
 ├─ Item(1)
 └─ Item(1)
```

원인
- Item 객체를 식별할 기준이 없음
- 같은 item_id라도 row마다 새 Item 생성

---

## 5. 정상 resultMap (부모 + 자식 `<id>` 모두 존재)

```xml
<resultMap id="orderMap" type="Order">
  <id property="orderId" column="order_id"/>

  <collection property="items" ofType="Item">
    <id property="itemId" column="item_id"/>
    <result property="itemName" column="item_name"/>
  </collection>
</resultMap>
```

결과

```
Order(1)
 └─ Item(1)
```

---

## 6. Java 코드로 복구 불가한 이유

- 이미 객체 그래프가 깨진 상태
- distinct / groupingBy 사용 시 데이터 유실
- 조립 단계에서만 해결 가능

---

## 7. 실무 규칙 요약

1. JOIN + resultMap → 부모 `<id>` 필수
2. collection 사용 → 자식 `<id>` 필수
3. `<id>` 하나라도 빠지면 100% 버그
4. 이 문제는 SQL도, Java도 아닌 **매핑 설계 문제**

---

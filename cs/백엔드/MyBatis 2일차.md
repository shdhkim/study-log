# MyBatis ResultMap · 동적 SQL 종합 정리

본 문서는 지금까지 다룬 MyBatis 핵심 주제를 하나의 Markdown 파일로 체계적으로 정리한 자료다.

---

## 1. ResultMap의 본질

### 1.1 resultMap이 하는 일

* SQL 결과(ResultSet)를 **객체 또는 Map으로 변환**
* 단순 매핑이 아니라 **객체 동일성(identity)**, **중복 제거**, **관계 조립(1:N)** 까지 관여

### 1.2 resultType vs resultMap

| 구분         | resultType | resultMap |
| ---------- | ---------- | --------- |
| 매핑 방식      | 자동         | 명시적       |
| id 사용      | 불가         | 가능        |
| collection | 불가         | 가능        |
| JOIN 처리    | 단순         | 정교        |

---

## 2. resultMap의 `<id>` 의미

### 2.1 `<id>`의 진짜 역할

* PK 표시용이 아님
* MyBatis 내부에서 **객체를 식별하는 기준(Identity Key)**
* 같은 id 값 → 같은 객체 재사용

### 2.2 `<id>`가 없으면 발생하는 문제

* 같은 데이터라도 **row마다 새 객체 생성**
* JOIN 시 **중복 객체 폭증**
* collection 구조 붕괴

---

## 3. `<collection>`과 `<id>`

### 3.1 collection의 역할

* 1:N 관계를 객체 그래프로 조립

### 3.2 collection 내부 `<id>`의 의미

* 자식 객체의 식별자
* **중복 제거 기준**

### 3.3 collection에 `<id>`가 없을 때

* 같은 자식이 row마다 새 객체로 생성
* 하위 데이터가 여러 객체로 분산
* 나중에 distinct로 복구 불가능

---

## 4. 왜 List distinct로 해결할 수 없는가

### 4.1 중복의 본질

* 중복은 "같은 엔티티가 여러 객체로 찢어진 상태"

### 4.2 distinct의 한계

* 객체 하나를 버리는 방식
* 버려진 객체에 담긴 하위 데이터도 함께 유실
* 구조적으로 복구 불가

결론: **중복 제거는 사후 처리로 해결할 수 없고, 조립 단계에서 해결해야 한다**

---

## 5. resultMap에서 Map을 쓰는 경우

### 5.1 `type="map"`의 의미

* 한 row = Map 하나
* 객체 동일성 개념 없음
* id, collection, association 의미 없음

### 5.2 Map을 쓰는 정상적인 경우

* 통계 / 리포트 / pivot
* 관리자 화면 조회
* 코드 테이블
* 컬럼 구조가 동적인 조회

### 5.3 Map을 쓰면 안 되는 경우

* 엔티티 개념이 있는 데이터
* 1:N 구조 조립
* 비즈니스 로직 대상 데이터

---

## 6. LISTAGG 패턴

### 6.1 LISTAGG의 성격

* 1:N 관계를 SQL 단계에서 문자열 1개로 접음
* 구조적 데이터가 아닌 **표현용 집계**

### 6.2 LISTAGG + Map 결과 형태

* 주문 1건당 row 1개
* item_names는 String

```text
{
  orderId: 1,
  itemNames: "키보드, 마우스, 모니터"
}
```

### 6.3 LISTAGG 사용 기준

* 화면 표시용
* 요약 API
* 엑셀 다운로드

### 6.4 LISTAGG 쓰면 안 되는 경우

* 다시 item 단위 접근이 필요한 경우
* 하위 정보(id, 가격, 옵션)가 필요한 경우

---

## 7. `#{}` vs `${}`

### 7.1 `#{}`

* PreparedStatement 바인딩
* SQL Injection 안전
* 값(value)에 사용

### 7.2 `${}`

* 문자열 치환
* SQL Injection 위험
* SQL 구조(컬럼명, ORDER BY)에만 제한적으로 사용

### 7.3 문자열 비교 규칙

```sql
WHERE name = #{name}          -- 올바름
WHERE name = ${name}          -- 위험
WHERE name = '${name}'        -- 매우 위험
```

---

## 8. 동적 SQL `<if>`

### 8.1 역할

* 조건이 true면 SQL 조각을 추가
* 여러 개 동시에 적용 가능

### 8.2 사용 예

```xml
<if test="status != null">
  AND status = #{status}
</if>
```

---

## 9. `<choose>` / `<when>` / `<otherwise>`

### 9.1 실행 규칙

* 위에서부터 순서대로 검사
* **첫 번째 true 하나만 실행**
* if–else if–else 구조

### 9.2 대표 용도

* 검색 타입 분기
* 정렬 방식 분기

```xml
<choose>
  <when test="searchType == 'NAME'">...</when>
  <when test="searchType == 'EMAIL'">...</when>
  <otherwise>...</otherwise>
</choose>
```

---

## 10. `<where>`

### 10.1 역할

* 조건이 있으면 WHERE 자동 추가
* 앞의 AND / OR 자동 제거

### 10.2 예제

```xml
<where>
  <if test="name != null">AND name = #{name}</if>
</where>
```

---

## 11. `<trim>`

### 11.1 역할

* SQL 문자열 앞/뒤 제어
* WHERE, SET, VALUES, AND, OR, 콤마 처리

### 11.2 대표 사용처

* UPDATE SET
* INSERT VALUES
* OR 조건 묶기

```xml
<trim prefix="SET" suffixOverrides=",">
  ...
</trim>
```

---

## 12. `<where>` + `<choose>` 조합 규칙

### 12.1 정석 구조

```xml
<where>
  <choose>
    <when>...</when>
    <when>...</when>
  </choose>
  <if>...</if>
</where>
```

### 12.2 실무 규칙 요약

* where: 외곽 구조
* choose: 분기
* if: 누적
* OR 조건: trim 사용

---

## 13. 전체 핵심 요약

1. resultMap의 `<id>`는 객체 동일성의 핵심
2. collection에는 반드시 `<id>` 필요
3. Map은 조립하지 않을 때만 사용
4. LISTAGG는 표현용 집계
5. 값은 무조건 `#{}`
6. 분기는 `<choose>`, 누적은 `<if>`
7. where는 자동 정리, trim은 범용 제어

---



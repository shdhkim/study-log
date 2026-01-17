# MyBatis 문법 상세 정리 

Spring Boot + Oracle 환경 기준  
MyBatis 문법 정리

---

## 1. MyBatis 개요

MyBatis는 SQL 중심 ORM 프레임워크이다.

- SQL을 직접 작성
- Java 객체와 SQL 결과를 매핑
- SQL 제어력이 매우 높음


### 전체 구조
```text
Controller
  ↓
Service
  ↓
Mapper Interface (Java)
  ↓
Mapper XML (SQL)
  ↓
DB
```

---

## 2. Mapper Interface

### 기본 형태
```java
@Mapper
public interface UserMapper {
    User selectUserById(Long userId);
}
```

### 파라미터 여러 개
```java
User selectUser(
    @Param("id") Long id,
    @Param("status") String status
);
```

XML에서는 `#{id}`, `#{status}`로 접근한다.

---

## 3. Mapper XML 기본 구조

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.example.mapper.UserMapper">
```

- namespace는 Mapper Interface의 패키지 포함 전체 경로와 반드시 일치해야 한다.

---

## 4. SELECT 문

### 단건 조회
```xml
<select id="selectUserById" resultType="User">
    SELECT *
    FROM USER
    WHERE USER_ID = #{userId}
</select>
```

### 다건 조회
```xml
<select id="selectAll" resultType="User">
    SELECT * FROM USER
</select>
```

### 파라미터 타입 지정 (선택)
- resultType 대신 resultMap을 쓰는 경우가 많지만, 단순 매핑이면 resultType이 편하다.
```xml
<select id="selectByStatus" parameterType="string" resultType="User">
    SELECT *
    FROM USER
    WHERE STATUS = #{status}
</select>
```

---

## 5. resultType vs resultMap (매핑)

### resultType
- 컬럼명과 Java 필드명이 동일(또는 camelCase 자동 변환 설정)할 때 사용

```xml
<select id="selectSimple" resultType="User">
    SELECT USER_ID AS userId, USER_NAME AS userName
    FROM USER
</select>
```

### resultMap (실무 핵심)
- 컬럼명 ≠ 필드명
- 조인 결과 DTO 매핑
- 중첩(association/collection) 매핑 등

```xml
<resultMap id="userMap" type="User">
    <id property="userId" column="USER_ID"/>
    <result property="userName" column="USER_NAME"/>
    <result property="createdAt" column="CREATED_AT"/>
</resultMap>

<select id="selectUser" resultMap="userMap">
    SELECT USER_ID, USER_NAME, CREATED_AT
    FROM USER
    WHERE USER_ID = #{userId}
</select>
```

---

## 6. INSERT / UPDATE / DELETE

### INSERT
```xml
<insert id="insertUser">
    INSERT INTO USER (USER_ID, USER_NAME)
    VALUES (#{userId}, #{userName})
</insert>
```

### UPDATE
```xml
<update id="updateUser">
    UPDATE USER
    SET USER_NAME = #{userName}
    WHERE USER_ID = #{userId}
</update>
```

### DELETE
```xml
<delete id="deleteUser">
    DELETE FROM USER
    WHERE USER_ID = #{userId}
</delete>
```

---

## 7. #{ } vs ${ } (매우 중요)

| 구분 | #{ } | ${ } |
|---|---|---|
| 처리 방식 | PreparedStatement 바인딩 | 문자열 치환 |
| SQL Injection | 안전 | 위험 |
| 주 사용처 | 값(리터럴) 바인딩 | 컬럼명/정렬/동적 SQL 조각 |

### 잘못된 예 (Injection 위험)
```sql
WHERE USER_NAME = ${name}
```

### 올바른 예
```sql
WHERE USER_NAME = #{name}
```

### ${} 사용 허용 예 (화이트리스트 필수)
```xml
ORDER BY ${sortColumn}
```

---

## 8. 동적 SQL (MyBatis 핵심)

### 8.1 if
```xml
<select id="searchUsers" resultType="User">
    SELECT *
    FROM USER
    <where>
        <if test="userName != null and userName != ''">
            AND USER_NAME = #{userName}
        </if>
        <if test="age != null">
            AND AGE = #{age}
        </if>
    </where>
</select>
```

### 8.2 where
- WHERE 자동 붙임
- AND/OR 선행 문제 처리

```xml
<where>
    <if test="age != null">
        AGE = #{age}
    </if>
</where>
```

### 8.3 choose (if-else)
```xml
<choose>
    <when test="type == 'A'">
        TYPE = 'A'
    </when>
    <otherwise>
        TYPE = 'B'
    </otherwise>
</choose>
```

### 8.4 foreach (IN 절)
```xml
<select id="selectByIds" resultType="User">
    SELECT *
    FROM USER
    WHERE USER_ID IN
    <foreach collection="ids" item="id" open="(" close=")" separator=",">
        #{id}
    </foreach>
</select>
```

### 8.5 set (UPDATE)
- 마지막 콤마 처리 자동으로 깔끔해짐

```xml
<update id="updateUserDynamic">
    UPDATE USER
    <set>
        <if test="userName != null">
            USER_NAME = #{userName},
        </if>
        <if test="age != null">
            AGE = #{age},
        </if>
        <if test="status != null">
            STATUS = #{status}
        </if>
    </set>
    WHERE USER_ID = #{userId}
</update>
```

### 8.6 trim (prefix/suffix 제어)
```xml
<select id="searchUsersTrim" resultType="User">
    SELECT *
    FROM USER
    <trim prefix="WHERE" prefixOverrides="AND|OR">
        <if test="userName != null">
            AND USER_NAME = #{userName}
        </if>
        <if test="age != null">
            AND AGE = #{age}
        </if>
    </trim>
</select>
```

---

## 9. <sql> / <include> (재사용 SQL)

### 9.1 <sql> 정의 (공통 컬럼)
```xml
<sql id="baseUserColumns">
    USER_ID,
    USER_NAME,
    CREATED_AT
</sql>
```

### 9.2 <include> 사용
```xml
<select id="selectUsers" resultType="User">
    SELECT
        <include refid="baseUserColumns"/>
    FROM USER
</select>
```

### 9.3 WHERE 절 공통화
```xml
<sql id="userSearchCondition">
    <if test="userName != null and userName != ''">
        AND USER_NAME = #{userName}
    </if>
    <if test="age != null">
        AND AGE = #{age}
    </if>
</sql>

<select id="searchUsersWithInclude" resultType="User">
    SELECT *
    FROM USER
    <where>
        <include refid="userSearchCondition"/>
    </where>
</select>
```

### 9.4 JOIN 절 공통화
```xml
<sql id="userJoinDept">
    LEFT JOIN DEPT D ON U.DEPT_ID = D.DEPT_ID
</sql>

<select id="selectUserWithDept" resultType="UserDeptDto">
    SELECT U.USER_ID, U.USER_NAME, D.DEPT_NAME
    FROM USER U
    <include refid="userJoinDept"/>
</select>
```

### 9.5 include 중첩
```xml
<sql id="userBase">
    <include refid="baseUserColumns"/>
</sql>
```

### include 사용 주의점
- SQL 문법 단위로만 분리 (SELECT/FROM 한 조각만 떼어내면 가독성이 오히려 떨어짐)
- WHERE 내부 include는 AND 기준으로 작성 (prefixOverrides로 처리하거나 where/trim에 넣기)
- 세미콜론 사용 금지

---

## 10. 파라미터 타입 정리

### 10.1 단일 값
```java
User select(Long id);
```

### 10.2 DTO
```java
User select(UserSearchCond cond);
```

### 10.3 Map
```java
User select(Map<String, Object> param);
```

### 10.4 List
```java
List<User> selectByIds(@Param("ids") List<Long> ids);
```

---

## 11. 자동 매핑 규칙 (언더스코어 → 카멜)

| DB 컬럼 | Java 필드 |
|---|---|
| USER_ID | userId |
| CREATED_AT | createdAt |

설정:
```yaml
mybatis:
  configuration:
    map-underscore-to-camel-case: true
```

---

## 12. Oracle 페이징

### 12.1 ROWNUM 방식
```sql
SELECT *
FROM (
  SELECT A.*, ROWNUM RN
  FROM (
    SELECT *
    FROM USER
    ORDER BY CREATED_AT DESC
  ) A
)
WHERE RN BETWEEN #{start} AND #{end}
```

### 12.2 12c+ OFFSET/FETCH (가능 시 권장)
```sql
SELECT *
FROM USER
ORDER BY CREATED_AT DESC
OFFSET #{offset} ROWS FETCH NEXT #{size} ROWS ONLY
```

---

## 13. 로그 및 운영 팁

### 13.1 SQL 로그 활성화
```yaml
mybatis:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

### 13.2 count 쿼리 분리
- 목록 조회 쿼리와 total count는 분리하는 편이 안정적 (성능/정확성)

### 13.3 Mapper XML 분리 전략
- 도메인 단위로 분리
- 조회/수정 Mapper 분리도 고려
- include는 "공통 컬럼/공통 조건/공통 조인" 정도로만 사용 (남용 금지)

### 13.4 ${} 사용 시 안전장치 예시
- ORDER BY 컬럼은 서버에서 화이트리스트로 검증 후 주입
- 클라이언트 입력값을 그대로 ${}로 쓰면 보안 사고 확률이 매우 높다

---

## 14. MyBatis vs JPA 요약

| 항목 | MyBatis | JPA |
|---|---|---|
| SQL 제어 | 매우 강함 | 제한적 |
| 러닝커브 | 낮음 | 높음 |
| 대용량 쿼리 | 유리 | 튜닝 필요 |


---

## 결론

- 동적 SQL + include를 적절히 쓰면 유지보수성이 크게 좋아진다
- SQL 이해도가 곧 실무 생산성이다

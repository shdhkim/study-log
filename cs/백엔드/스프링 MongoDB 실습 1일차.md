# 03. Spring Boot MongoDB 기초 실습: Document CRUD 작성

## 1. 목표

이 파일에서는 Spring Boot에서 MongoDB를 연결하고 기본 CRUD API를 만든다.

실습 내용:

- MongoDB 실행
- Spring Boot MongoDB 연결
- `@Document` 기반 도메인 작성
- `MongoRepository` 사용
- 등록, 조회, 목록, 수정, 삭제 API 작성
- MongoDB Document 구조 이해

---

## 2. MongoDB란

MongoDB는 Document 기반 NoSQL 데이터베이스다. RDBMS의 테이블/행 구조와 달리 JSON과 비슷한 BSON Document 형태로 데이터를 저장한다.

RDBMS와 비교하면 다음과 같다.

| RDBMS    | MongoDB                   |
| -------- | ------------------------- |
| Database | Database                  |
| Table    | Collection                |
| Row      | Document                  |
| Column   | Field                     |
| PK       | `_id`                     |
| SQL      | Query Method, Mongo Query |

예시 Document:

```json
{
  "_id": "662f1c...",
  "title": "Spring MongoDB",
  "content": "MongoDB practice",
  "writer": "kim",
  "viewCount": 0
}
```

---

## 3. MongoDB 실행

### 3.1 Docker로 MongoDB 실행

```bash
docker run --name mongo-practice \
  -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=root \
  -e MONGO_INITDB_ROOT_PASSWORD=1234 \
  -d mongo:7
```

실행 확인:

```bash
docker ps
```

MongoDB Shell 접속:

```bash
docker exec -it mongo-practice mongosh -u root -p 1234
```

DB 목록 확인:

```javascript
show dbs
```

---

## 4. 의존성

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.3.0'
    id 'io.spring.dependency-management' version '1.1.5'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'

java {
    sourceCompatibility = '17'
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-mongodb'
    implementation 'org.springframework.boot:spring-boot-starter-validation'

    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

---

## 5. application.yml

```yaml
spring:
  data:
    mongodb:
      uri: mongodb://root:1234@localhost:27017/mongo_practice?authSource=admin

server:
  port: 8080
```

설명:

| 설정               | 의미                               |
| ------------------ | ---------------------------------- |
| `root`             | MongoDB 사용자                     |
| `1234`             | MongoDB 비밀번호                   |
| `localhost:27017`  | MongoDB 주소                       |
| `mongo_practice`   | 사용할 DB 이름                     |
| `authSource=admin` | 인증 계정이 admin DB에 있음을 의미 |

---

## 6. 패키지 구조

```text
src/main/java/com/example/mongobasic
 ├─ MongoBasicApplication.java
 ├─ domain
 │   └─ Board.java
 ├─ repository
 │   └─ BoardRepository.java
 ├─ service
 │   └─ BoardService.java
 ├─ controller
 │   └─ BoardController.java
 └─ dto
     ├─ BoardCreateRequest.java
     ├─ BoardUpdateRequest.java
     └─ BoardResponse.java
```

---

## 7. 메인 클래스

```java
package com.example.mongobasic;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MongoBasicApplication {

    public static void main(String[] args) {
        SpringApplication.run(MongoBasicApplication.class, args);
    }
}
```

---

## 8. Domain 작성

```java
package com.example.mongobasic.domain;

import lombok.AccessLevel;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

import java.time.LocalDateTime;

@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Document(collection = "boards")
public class Board {

    @Id
    private String id;

    private String title;

    private String content;

    private String writer;

    private long viewCount;

    private LocalDateTime createdAt;

    private LocalDateTime updatedAt;

    @Builder
    public Board(String title, String content, String writer) {
        this.title = title;
        this.content = content;
        this.writer = writer;
        this.viewCount = 0;
        this.createdAt = LocalDateTime.now();
        this.updatedAt = LocalDateTime.now();
    }

    public void update(String title, String content) {
        this.title = title;
        this.content = content;
        this.updatedAt = LocalDateTime.now();
    }

    public void increaseViewCount() {
        this.viewCount++;
        this.updatedAt = LocalDateTime.now();
    }
}
```

핵심 어노테이션:

| 어노테이션                         | 의미                    |
| ---------------------------------- | ----------------------- |
| `@Document(collection = "boards")` | MongoDB Collection 매핑 |
| `@Id`                              | MongoDB `_id` 필드 매핑 |

MongoDB의 `_id`는 보통 ObjectId로 생성되지만 Spring에서는 문자열로 받아도 된다.

---

## 9. DTO 작성

### 9.1 BoardCreateRequest

```java
package com.example.mongobasic.dto;

import jakarta.validation.constraints.NotBlank;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Getter
@NoArgsConstructor
public class BoardCreateRequest {

    @NotBlank
    private String title;

    @NotBlank
    private String content;

    @NotBlank
    private String writer;
}
```

---

### 9.2 BoardUpdateRequest

```java
package com.example.mongobasic.dto;

import jakarta.validation.constraints.NotBlank;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Getter
@NoArgsConstructor
public class BoardUpdateRequest {

    @NotBlank
    private String title;

    @NotBlank
    private String content;
}
```

---

### 9.3 BoardResponse

```java
package com.example.mongobasic.dto;

import com.example.mongobasic.domain.Board;
import lombok.Getter;

import java.time.LocalDateTime;

@Getter
public class BoardResponse {

    private final String id;
    private final String title;
    private final String content;
    private final String writer;
    private final long viewCount;
    private final LocalDateTime createdAt;
    private final LocalDateTime updatedAt;

    public BoardResponse(Board board) {
        this.id = board.getId();
        this.title = board.getTitle();
        this.content = board.getContent();
        this.writer = board.getWriter();
        this.viewCount = board.getViewCount();
        this.createdAt = board.getCreatedAt();
        this.updatedAt = board.getUpdatedAt();
    }
}
```

---

## 10. Repository 작성

```java
package com.example.mongobasic.repository;

import com.example.mongobasic.domain.Board;
import org.springframework.data.mongodb.repository.MongoRepository;

import java.util.List;

public interface BoardRepository extends MongoRepository<Board, String> {

    List<Board> findByWriter(String writer);

    List<Board> findByTitleContaining(String keyword);
}
```

`MongoRepository<Board, String>`의 의미:

| 타입     | 의미                   |
| -------- | ---------------------- |
| `Board`  | 저장할 Document 클래스 |
| `String` | ID 타입                |

Spring Data MongoDB는 메서드 이름으로 쿼리를 자동 생성할 수 있다.

| 메서드                  | 의미                                   |
| ----------------------- | -------------------------------------- |
| `findByWriter`          | writer 필드가 일치하는 Document 조회   |
| `findByTitleContaining` | title에 keyword가 포함된 Document 조회 |

---

## 11. Service 작성

```java
package com.example.mongobasic.service;

import com.example.mongobasic.domain.Board;
import com.example.mongobasic.dto.BoardCreateRequest;
import com.example.mongobasic.dto.BoardResponse;
import com.example.mongobasic.dto.BoardUpdateRequest;
import com.example.mongobasic.repository.BoardRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
@RequiredArgsConstructor
public class BoardService {

    private final BoardRepository boardRepository;

    public BoardResponse create(BoardCreateRequest request) {
        Board board = Board.builder()
                .title(request.getTitle())
                .content(request.getContent())
                .writer(request.getWriter())
                .build();

        Board savedBoard = boardRepository.save(board);
        return new BoardResponse(savedBoard);
    }

    public BoardResponse findById(String id) {
        Board board = boardRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("게시글을 찾을 수 없습니다. id=" + id));

        board.increaseViewCount();
        Board savedBoard = boardRepository.save(board);

        return new BoardResponse(savedBoard);
    }

    public List<BoardResponse> findAll() {
        return boardRepository.findAll()
                .stream()
                .map(BoardResponse::new)
                .toList();
    }

    public List<BoardResponse> findByWriter(String writer) {
        return boardRepository.findByWriter(writer)
                .stream()
                .map(BoardResponse::new)
                .toList();
    }

    public List<BoardResponse> searchByTitle(String keyword) {
        return boardRepository.findByTitleContaining(keyword)
                .stream()
                .map(BoardResponse::new)
                .toList();
    }

    public BoardResponse update(String id, BoardUpdateRequest request) {
        Board board = boardRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("게시글을 찾을 수 없습니다. id=" + id));

        board.update(request.getTitle(), request.getContent());
        Board savedBoard = boardRepository.save(board);

        return new BoardResponse(savedBoard);
    }

    public void delete(String id) {
        if (!boardRepository.existsById(id)) {
            throw new IllegalArgumentException("게시글을 찾을 수 없습니다. id=" + id);
        }

        boardRepository.deleteById(id);
    }
}
```

MongoDB에서 `save()`는 다음처럼 동작한다.

| 상황                   | 동작                |
| ---------------------- | ------------------- |
| `_id`가 없는 새 객체   | insert              |
| `_id`가 이미 있는 객체 | replace/update 성격 |

---

## 12. Controller 작성

```java
package com.example.mongobasic.controller;

import com.example.mongobasic.dto.BoardCreateRequest;
import com.example.mongobasic.dto.BoardResponse;
import com.example.mongobasic.dto.BoardUpdateRequest;
import com.example.mongobasic.service.BoardService;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

import java.util.List;
import java.util.Map;

@RestController
@RequiredArgsConstructor
@RequestMapping("/api/boards")
public class BoardController {

    private final BoardService boardService;

    @PostMapping
    public BoardResponse create(@Valid @RequestBody BoardCreateRequest request) {
        return boardService.create(request);
    }

    @GetMapping("/{id}")
    public BoardResponse findById(@PathVariable String id) {
        return boardService.findById(id);
    }

    @GetMapping
    public List<BoardResponse> findAll() {
        return boardService.findAll();
    }

    @GetMapping("/writer/{writer}")
    public List<BoardResponse> findByWriter(@PathVariable String writer) {
        return boardService.findByWriter(writer);
    }

    @GetMapping("/search")
    public List<BoardResponse> searchByTitle(@RequestParam String keyword) {
        return boardService.searchByTitle(keyword);
    }

    @PutMapping("/{id}")
    public BoardResponse update(
            @PathVariable String id,
            @Valid @RequestBody BoardUpdateRequest request
    ) {
        return boardService.update(id, request);
    }

    @DeleteMapping("/{id}")
    public Map<String, Object> delete(@PathVariable String id) {
        boardService.delete(id);
        return Map.of("message", "deleted", "id", id);
    }
}
```

---

## 13. API 테스트

### 13.1 게시글 등록

```bash
curl -X POST http://localhost:8080/api/boards \
  -H "Content-Type: application/json" \
  -d '{
    "title":"Spring MongoDB 실습",
    "content":"MongoDB CRUD를 연습합니다.",
    "writer":"kim"
  }'
```

응답 예시:

```json
{
  "id": "662f3b1a8e9e5c1b6f5c1111",
  "title": "Spring MongoDB 실습",
  "content": "MongoDB CRUD를 연습합니다.",
  "writer": "kim",
  "viewCount": 0,
  "createdAt": "2026-04-26T16:00:00",
  "updatedAt": "2026-04-26T16:00:00"
}
```

---

### 13.2 전체 조회

```bash
curl http://localhost:8080/api/boards
```

---

### 13.3 단건 조회

```bash
curl http://localhost:8080/api/boards/게시글ID
```

조회할 때마다 `viewCount`가 증가한다.

---

### 13.4 작성자 조회

```bash
curl http://localhost:8080/api/boards/writer/kim
```

---

### 13.5 제목 검색

```bash
curl "http://localhost:8080/api/boards/search?keyword=MongoDB"
```

---

### 13.6 게시글 수정

```bash
curl -X PUT http://localhost:8080/api/boards/게시글ID \
  -H "Content-Type: application/json" \
  -d '{
    "title":"수정된 제목",
    "content":"수정된 내용"
  }'
```

---

### 13.7 게시글 삭제

```bash
curl -X DELETE http://localhost:8080/api/boards/게시글ID
```

---

## 14. MongoDB Shell로 데이터 확인

MongoDB Shell 접속:

```bash
docker exec -it mongo-practice mongosh -u root -p 1234
```

DB 선택:

```javascript
use mongo_practice
```

Collection 목록:

```javascript
show collections
```

Document 조회:

```javascript
db.boards.find();
```

보기 좋게 출력:

```javascript
db.boards.find().pretty();
```

작성자 조건 조회:

```javascript
db.boards.find({ writer: "kim" }).pretty();
```

삭제:

```javascript
db.boards.deleteOne({ writer: "kim" });
```

---

## 15. RDBMS와 다른 점

### 15.1 스키마가 상대적으로 자유롭다

MongoDB는 Collection 안의 Document들이 완전히 동일한 필드 구조를 가질 필요가 없다.

```json
{ "title": "A", "writer": "kim" }
{ "title": "B", "writer": "lee", "tags": ["spring", "mongo"] }
```

하지만 실무에서는 너무 자유롭게 쓰면 관리가 어려워진다. 애플리케이션 레벨에서 DTO와 Domain 구조를 명확히 잡는 것이 좋다.

---

### 15.2 조인보다 Document 중심 설계

MongoDB는 RDBMS처럼 조인을 중심으로 설계하지 않는다. 자주 같이 조회되는 데이터는 하나의 Document 안에 포함시키는 방식도 많이 사용한다.

---

### 15.3 트랜잭션은 가능하지만 신중히 사용

MongoDB도 트랜잭션을 지원하지만, MongoDB의 장점은 보통 단일 Document 중심의 빠른 읽기/쓰기에서 나온다. 복잡한 다중 Collection 트랜잭션이 많다면 RDBMS가 더 적합할 수 있다.

---

## 16. 정리

| 항목              | 핵심                             |
| ----------------- | -------------------------------- |
| `@Document`       | MongoDB Collection과 클래스 매핑 |
| `@Id`             | `_id` 필드 매핑                  |
| `MongoRepository` | 기본 CRUD 제공                   |
| `save()`          | insert 또는 update               |
| Query Method      | 메서드 이름으로 쿼리 자동 생성   |
| Collection        | RDBMS의 Table과 유사             |
| Document          | RDBMS의 Row와 유사               |

MongoDB는 게시글, 로그, 설정, 이벤트성 데이터, 유연한 구조의 데이터 저장에 실습하기 좋다.

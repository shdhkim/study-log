# 04. Spring Boot MongoDB 심화 실습: Embedded Document, Query, Index, Pagination

## 1. 목표

이 파일에서는 MongoDB의 실무적인 사용 방식을 조금 더 깊게 다룬다.

실습 내용:

- Embedded Document 설계
- 댓글이 포함된 게시글 구조 작성
- `MongoTemplate`으로 동적 쿼리 작성
- 페이징 처리
- 정렬 처리
- Index 설정
- 부분 업데이트
- MongoDB 설계 시 주의사항

---

## 2. 예제 도메인

게시글과 댓글을 예제로 사용한다.

RDBMS라면 보통 다음처럼 나눈다.

```text
boards table
comments table
```

MongoDB에서는 댓글을 게시글 Document 안에 포함할 수 있다.

```json
{
  "_id": "...",
  "title": "MongoDB 심화",
  "writer": "kim",
  "comments": [
    { "commentId": "...", "writer": "lee", "content": "좋은 글입니다." },
    { "commentId": "...", "writer": "park", "content": "감사합니다." }
  ]
}
```

이런 구조를 Embedded Document라고 한다.

---

## 3. 언제 Embedded Document를 쓰는가

적합한 경우:

| 상황                                 | 설명                                               |
| ------------------------------------ | -------------------------------------------------- |
| 부모와 자식이 항상 같이 조회됨       | 게시글 상세 조회 시 댓글 일부도 같이 보여주는 경우 |
| 자식 데이터 크기가 제한적임          | 댓글 수가 무한히 커지지 않는 경우                  |
| 자식이 독립적으로 자주 조회되지 않음 | 댓글만 따로 검색하는 요구가 약한 경우              |

부적합한 경우:

| 상황                        | 이유                             |
| --------------------------- | -------------------------------- |
| 댓글 수가 매우 많음         | Document 크기가 커짐             |
| 댓글만 독립적으로 자주 검색 | 별도 Collection이 유리할 수 있음 |
| 댓글 수정/삭제가 매우 빈번  | 배열 내부 업데이트 복잡도 증가   |

MongoDB Document 하나의 최대 크기는 제한이 있으므로, 무한히 커지는 배열은 피해야 한다.

---

## 4. 의존성

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-mongodb'
    implementation 'org.springframework.boot:spring-boot-starter-validation'

    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

---

## 5. application.yml

```yaml
spring:
  data:
    mongodb:
      uri: mongodb://root:1234@localhost:27017/mongo_advanced?authSource=admin

server:
  port: 8080
```

---

## 6. 패키지 구조

```text
src/main/java/com/example/mongoadvanced
 ├─ MongoAdvancedApplication.java
 ├─ domain
 │   ├─ Article.java
 │   └─ Comment.java
 ├─ repository
 │   └─ ArticleRepository.java
 ├─ service
 │   └─ ArticleService.java
 ├─ controller
 │   └─ ArticleController.java
 └─ dto
     ├─ ArticleCreateRequest.java
     ├─ ArticleUpdateRequest.java
     ├─ ArticleSearchCondition.java
     ├─ CommentCreateRequest.java
     ├─ ArticleResponse.java
     └─ CommentResponse.java
```

---

## 7. 메인 클래스

```java
package com.example.mongoadvanced;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MongoAdvancedApplication {

    public static void main(String[] args) {
        SpringApplication.run(MongoAdvancedApplication.class, args);
    }
}
```

---

## 8. Domain 작성

### 8.1 Comment

```java
package com.example.mongoadvanced.domain;

import lombok.AccessLevel;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

import java.time.LocalDateTime;
import java.util.UUID;

@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Comment {

    private String commentId;
    private String writer;
    private String content;
    private LocalDateTime createdAt;

    @Builder
    public Comment(String writer, String content) {
        this.commentId = UUID.randomUUID().toString();
        this.writer = writer;
        this.content = content;
        this.createdAt = LocalDateTime.now();
    }
}
```

`Comment`는 별도 Collection이 아니라 `Article` 안에 포함될 Embedded Document다.

---

### 8.2 Article

```java
package com.example.mongoadvanced.domain;

import lombok.AccessLevel;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.index.CompoundIndex;
import org.springframework.data.mongodb.core.index.Indexed;
import org.springframework.data.mongodb.core.mapping.Document;

import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;

@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Document(collection = "articles")
@CompoundIndex(name = "idx_writer_createdAt", def = "{ 'writer': 1, 'createdAt': -1 }")
public class Article {

    @Id
    private String id;

    @Indexed
    private String title;

    private String content;

    @Indexed
    private String writer;

    private List<String> tags = new ArrayList<>();

    private List<Comment> comments = new ArrayList<>();

    private long viewCount;

    private LocalDateTime createdAt;

    private LocalDateTime updatedAt;

    @Builder
    public Article(String title, String content, String writer, List<String> tags) {
        this.title = title;
        this.content = content;
        this.writer = writer;
        this.tags = tags == null ? new ArrayList<>() : tags;
        this.comments = new ArrayList<>();
        this.viewCount = 0;
        this.createdAt = LocalDateTime.now();
        this.updatedAt = LocalDateTime.now();
    }

    public void update(String title, String content, List<String> tags) {
        this.title = title;
        this.content = content;
        this.tags = tags == null ? new ArrayList<>() : tags;
        this.updatedAt = LocalDateTime.now();
    }

    public void addComment(Comment comment) {
        this.comments.add(comment);
        this.updatedAt = LocalDateTime.now();
    }

    public void increaseViewCount() {
        this.viewCount++;
        this.updatedAt = LocalDateTime.now();
    }
}
```

인덱스 설명:

| 어노테이션       | 의미                  |
| ---------------- | --------------------- |
| `@Indexed`       | 단일 필드 인덱스 생성 |
| `@CompoundIndex` | 복합 인덱스 생성      |

복합 인덱스:

```java
@CompoundIndex(name = "idx_writer_createdAt", def = "{ 'writer': 1, 'createdAt': -1 }")
```

의미:

```text
writer 오름차순 + createdAt 내림차순 기준 인덱스
```

작성자별 최신 글 조회에 유리하다.

---

## 9. DTO 작성

### 9.1 ArticleCreateRequest

```java
package com.example.mongoadvanced.dto;

import jakarta.validation.constraints.NotBlank;
import lombok.Getter;
import lombok.NoArgsConstructor;

import java.util.List;

@Getter
@NoArgsConstructor
public class ArticleCreateRequest {

    @NotBlank
    private String title;

    @NotBlank
    private String content;

    @NotBlank
    private String writer;

    private List<String> tags;
}
```

---

### 9.2 ArticleUpdateRequest

```java
package com.example.mongoadvanced.dto;

import jakarta.validation.constraints.NotBlank;
import lombok.Getter;
import lombok.NoArgsConstructor;

import java.util.List;

@Getter
@NoArgsConstructor
public class ArticleUpdateRequest {

    @NotBlank
    private String title;

    @NotBlank
    private String content;

    private List<String> tags;
}
```

---

### 9.3 CommentCreateRequest

```java
package com.example.mongoadvanced.dto;

import jakarta.validation.constraints.NotBlank;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Getter
@NoArgsConstructor
public class CommentCreateRequest {

    @NotBlank
    private String writer;

    @NotBlank
    private String content;
}
```

---

### 9.4 ArticleSearchCondition

```java
package com.example.mongoadvanced.dto;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class ArticleSearchCondition {

    private String title;
    private String writer;
    private String tag;
}
```

검색 조건은 `@ModelAttribute`로 받을 것이므로 setter를 열어둔다.

---

### 9.5 CommentResponse

```java
package com.example.mongoadvanced.dto;

import com.example.mongoadvanced.domain.Comment;
import lombok.Getter;

import java.time.LocalDateTime;

@Getter
public class CommentResponse {

    private final String commentId;
    private final String writer;
    private final String content;
    private final LocalDateTime createdAt;

    public CommentResponse(Comment comment) {
        this.commentId = comment.getCommentId();
        this.writer = comment.getWriter();
        this.content = comment.getContent();
        this.createdAt = comment.getCreatedAt();
    }
}
```

---

### 9.6 ArticleResponse

```java
package com.example.mongoadvanced.dto;

import com.example.mongoadvanced.domain.Article;
import lombok.Getter;

import java.time.LocalDateTime;
import java.util.List;

@Getter
public class ArticleResponse {

    private final String id;
    private final String title;
    private final String content;
    private final String writer;
    private final List<String> tags;
    private final List<CommentResponse> comments;
    private final long viewCount;
    private final LocalDateTime createdAt;
    private final LocalDateTime updatedAt;

    public ArticleResponse(Article article) {
        this.id = article.getId();
        this.title = article.getTitle();
        this.content = article.getContent();
        this.writer = article.getWriter();
        this.tags = article.getTags();
        this.comments = article.getComments()
                .stream()
                .map(CommentResponse::new)
                .toList();
        this.viewCount = article.getViewCount();
        this.createdAt = article.getCreatedAt();
        this.updatedAt = article.getUpdatedAt();
    }
}
```

---

## 10. Repository 작성

```java
package com.example.mongoadvanced.repository;

import com.example.mongoadvanced.domain.Article;
import org.springframework.data.mongodb.repository.MongoRepository;

import java.util.List;

public interface ArticleRepository extends MongoRepository<Article, String> {

    List<Article> findByWriterOrderByCreatedAtDesc(String writer);

    List<Article> findByTagsContaining(String tag);
}
```

---

## 11. Service 작성: Repository + MongoTemplate 같이 사용

동적 검색 조건은 `MongoTemplate`이 편하다.

```java
package com.example.mongoadvanced.service;

import com.example.mongoadvanced.domain.Article;
import com.example.mongoadvanced.domain.Comment;
import com.example.mongoadvanced.dto.*;
import com.example.mongoadvanced.repository.ArticleRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageImpl;
import org.springframework.data.domain.Pageable;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.data.mongodb.core.query.Update;
import org.springframework.stereotype.Service;
import org.springframework.util.StringUtils;

import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;

@Service
@RequiredArgsConstructor
public class ArticleService {

    private final ArticleRepository articleRepository;
    private final MongoTemplate mongoTemplate;

    public ArticleResponse create(ArticleCreateRequest request) {
        Article article = Article.builder()
                .title(request.getTitle())
                .content(request.getContent())
                .writer(request.getWriter())
                .tags(request.getTags())
                .build();

        Article savedArticle = articleRepository.save(article);
        return new ArticleResponse(savedArticle);
    }

    public ArticleResponse findById(String id) {
        Article article = articleRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("게시글을 찾을 수 없습니다. id=" + id));

        article.increaseViewCount();
        Article savedArticle = articleRepository.save(article);

        return new ArticleResponse(savedArticle);
    }

    public Page<ArticleResponse> search(ArticleSearchCondition condition, Pageable pageable) {
        Query query = createSearchQuery(condition);

        long total = mongoTemplate.count(query, Article.class);

        query.with(pageable);

        List<ArticleResponse> content = mongoTemplate.find(query, Article.class)
                .stream()
                .map(ArticleResponse::new)
                .toList();

        return new PageImpl<>(content, pageable, total);
    }

    public ArticleResponse update(String id, ArticleUpdateRequest request) {
        Article article = articleRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("게시글을 찾을 수 없습니다. id=" + id));

        article.update(request.getTitle(), request.getContent(), request.getTags());
        Article savedArticle = articleRepository.save(article);

        return new ArticleResponse(savedArticle);
    }

    public void updateTitleOnly(String id, String title) {
        Query query = Query.query(Criteria.where("id").is(id));
        Update update = new Update()
                .set("title", title)
                .set("updatedAt", LocalDateTime.now());

        mongoTemplate.updateFirst(query, update, Article.class);
    }

    public ArticleResponse addComment(String articleId, CommentCreateRequest request) {
        Article article = articleRepository.findById(articleId)
                .orElseThrow(() -> new IllegalArgumentException("게시글을 찾을 수 없습니다. id=" + articleId));

        Comment comment = Comment.builder()
                .writer(request.getWriter())
                .content(request.getContent())
                .build();

        article.addComment(comment);
        Article savedArticle = articleRepository.save(article);

        return new ArticleResponse(savedArticle);
    }

    public void delete(String id) {
        articleRepository.deleteById(id);
    }

    private Query createSearchQuery(ArticleSearchCondition condition) {
        List<Criteria> criteriaList = new ArrayList<>();

        if (StringUtils.hasText(condition.getTitle())) {
            criteriaList.add(Criteria.where("title").regex(condition.getTitle(), "i"));
        }

        if (StringUtils.hasText(condition.getWriter())) {
            criteriaList.add(Criteria.where("writer").is(condition.getWriter()));
        }

        if (StringUtils.hasText(condition.getTag())) {
            criteriaList.add(Criteria.where("tags").is(condition.getTag()));
        }

        Query query = new Query();

        if (!criteriaList.isEmpty()) {
            query.addCriteria(new Criteria().andOperator(criteriaList.toArray(new Criteria[0])));
        }

        return query;
    }
}
```

---

## 12. MongoTemplate 핵심 설명

### 12.1 Query

```java
Query query = new Query();
query.addCriteria(Criteria.where("writer").is("kim"));
```

의미:

```javascript
db.articles.find({ writer: "kim" });
```

---

### 12.2 Regex 검색

```java
Criteria.where("title").regex(condition.getTitle(), "i")
```

의미:

```javascript
db.articles.find({ title: /검색어/i });
```

`i`는 대소문자 무시 옵션이다.

주의할 점:

- 정규식 검색은 인덱스를 효율적으로 못 탈 수 있다.
- 검색 기능이 중요하면 MongoDB Text Index나 검색 엔진을 고려한다.

---

### 12.3 배열 필드 검색

```java
Criteria.where("tags").is("spring")
```

의미:

```javascript
db.articles.find({ tags: "spring" });
```

`tags` 배열 안에 `spring`이 포함된 Document를 찾는다.

---

### 12.4 부분 업데이트

```java
Update update = new Update()
        .set("title", title)
        .set("updatedAt", LocalDateTime.now());

mongoTemplate.updateFirst(query, update, Article.class);
```

Repository의 `save()`는 객체 전체를 저장하는 느낌이고, `MongoTemplate`의 `Update`는 특정 필드만 수정할 수 있다.

---

## 13. Controller 작성

```java
package com.example.mongoadvanced.controller;

import com.example.mongoadvanced.dto.*;
import com.example.mongoadvanced.service.ArticleService;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.web.bind.annotation.*;

import java.util.Map;

@RestController
@RequiredArgsConstructor
@RequestMapping("/api/articles")
public class ArticleController {

    private final ArticleService articleService;

    @PostMapping
    public ArticleResponse create(@Valid @RequestBody ArticleCreateRequest request) {
        return articleService.create(request);
    }

    @GetMapping("/{id}")
    public ArticleResponse findById(@PathVariable String id) {
        return articleService.findById(id);
    }

    @GetMapping
    public Page<ArticleResponse> search(
            @ModelAttribute ArticleSearchCondition condition,
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size
    ) {
        Pageable pageable = PageRequest.of(
                page,
                size,
                Sort.by(Sort.Direction.DESC, "createdAt")
        );

        return articleService.search(condition, pageable);
    }

    @PutMapping("/{id}")
    public ArticleResponse update(
            @PathVariable String id,
            @Valid @RequestBody ArticleUpdateRequest request
    ) {
        return articleService.update(id, request);
    }

    @PatchMapping("/{id}/title")
    public Map<String, Object> updateTitleOnly(
            @PathVariable String id,
            @RequestParam String title
    ) {
        articleService.updateTitleOnly(id, title);
        return Map.of("message", "title updated", "id", id);
    }

    @PostMapping("/{id}/comments")
    public ArticleResponse addComment(
            @PathVariable String id,
            @Valid @RequestBody CommentCreateRequest request
    ) {
        return articleService.addComment(id, request);
    }

    @DeleteMapping("/{id}")
    public Map<String, Object> delete(@PathVariable String id) {
        articleService.delete(id);
        return Map.of("message", "deleted", "id", id);
    }
}
```

---

## 14. API 테스트

### 14.1 게시글 등록

```bash
curl -X POST http://localhost:8080/api/articles \
  -H "Content-Type: application/json" \
  -d '{
    "title":"MongoDB 심화 실습",
    "content":"Embedded Document와 MongoTemplate을 실습합니다.",
    "writer":"kim",
    "tags":["spring", "mongodb", "backend"]
  }'
```

---

### 14.2 댓글 추가

```bash
curl -X POST http://localhost:8080/api/articles/게시글ID/comments \
  -H "Content-Type: application/json" \
  -d '{
    "writer":"lee",
    "content":"좋은 글입니다."
  }'
```

---

### 14.3 조건 검색

작성자 검색:

```bash
curl "http://localhost:8080/api/articles?writer=kim"
```

태그 검색:

```bash
curl "http://localhost:8080/api/articles?tag=spring"
```

제목 검색:

```bash
curl "http://localhost:8080/api/articles?title=Mongo"
```

페이징:

```bash
curl "http://localhost:8080/api/articles?page=0&size=5"
```

조건 + 페이징:

```bash
curl "http://localhost:8080/api/articles?writer=kim&tag=mongodb&page=0&size=10"
```

---

### 14.4 제목만 부분 수정

```bash
curl -X PATCH "http://localhost:8080/api/articles/게시글ID/title?title=변경된제목"
```

---

### 14.5 삭제

```bash
curl -X DELETE http://localhost:8080/api/articles/게시글ID
```

---

## 15. MongoDB Shell 확인

접속:

```bash
docker exec -it mongo-practice mongosh -u root -p 1234
```

DB 선택:

```javascript
use mongo_advanced
```

전체 조회:

```javascript
db.articles.find().pretty();
```

작성자 조회:

```javascript
db.articles.find({ writer: "kim" }).pretty();
```

태그 조회:

```javascript
db.articles.find({ tags: "spring" }).pretty();
```

댓글 작성자 조회:

```javascript
db.articles.find({ "comments.writer": "lee" }).pretty();
```

인덱스 확인:

```javascript
db.articles.getIndexes();
```

실행 계획 확인:

```javascript
db.articles.find({ writer: "kim" }).explain("executionStats");
```

---

## 16. 인덱스 실무 주의사항

### 16.1 인덱스는 조회를 빠르게 하지만 쓰기 비용이 증가한다

인덱스를 많이 만들면 조회는 빨라질 수 있지만 insert/update/delete 시 인덱스도 함께 갱신해야 하므로 쓰기 비용이 증가한다.

---

### 16.2 자주 검색하는 조건에 인덱스를 건다

예를 들어 다음 검색이 많다면:

```text
writer = ? order by createdAt desc
```

복합 인덱스가 유리하다.

```java
@CompoundIndex(name = "idx_writer_createdAt", def = "{ 'writer': 1, 'createdAt': -1 }")
```

---

### 16.3 정규식 검색은 주의한다

```java
Criteria.where("title").regex(keyword, "i")
```

이 방식은 편하지만 대량 데이터에서 성능 문제가 생길 수 있다. 검색 요구가 강하면 다음을 고려한다.

- MongoDB Text Index
- Elasticsearch/OpenSearch
- 별도 검색 테이블 또는 검색용 Collection

---

## 17. Embedded Document 설계 주의사항

### 17.1 배열이 무한히 커지면 안 된다

댓글을 게시글 안에 계속 넣으면 한 Document가 너무 커질 수 있다. 댓글이 매우 많아질 수 있는 서비스라면 댓글을 별도 Collection으로 분리하는 것이 좋다.

---

### 17.2 함께 조회되는 데이터인지 확인한다

Embedded Document는 같이 조회할 때 유리하다.

```text
게시글 상세 + 최근 댓글 3개
```

이런 요구에는 적합할 수 있다.

반대로 댓글만 따로 관리하고 검색하는 요구가 강하면 분리하는 것이 좋다.

---

## 18. Repository와 MongoTemplate 선택 기준

| 방식              | 장점                            | 적합한 상황                   |
| ----------------- | ------------------------------- | ----------------------------- |
| `MongoRepository` | 간단하고 빠르게 CRUD 작성 가능  | 기본 CRUD, 단순 조건 조회     |
| `MongoTemplate`   | 동적 쿼리, 부분 업데이트에 강함 | 복잡한 검색 조건, Update 쿼리 |

실무에서는 둘 중 하나만 쓰기보다 함께 사용하는 경우가 많다.

---

## 19. 정리

| 주제              | 핵심                                    |
| ----------------- | --------------------------------------- |
| Embedded Document | 관련 데이터를 하나의 Document 안에 포함 |
| `MongoTemplate`   | 동적 쿼리와 부분 업데이트에 유리        |
| `Criteria`        | MongoDB 조건식 작성                     |
| `Query`           | 조회 쿼리 객체                          |
| `Update`          | 부분 수정 객체                          |
| `Pageable`        | 페이징, 정렬 처리                       |
| `@Indexed`        | 단일 인덱스                             |
| `@CompoundIndex`  | 복합 인덱스                             |

MongoDB는 RDBMS처럼 정규화 중심으로만 접근하면 장점을 살리기 어렵다. 조회 패턴을 기준으로 Document 구조를 설계하는 것이 중요하다.

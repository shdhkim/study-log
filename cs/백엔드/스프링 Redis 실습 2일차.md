# 02. Spring Boot Redis 실습: 객체 JSON 캐싱과 Spring Cache 적용

## 1. 목표

이 파일에서는 Redis를 단순 문자열 저장소가 아니라 캐시 저장소로 사용하는 방법을 실습한다.

실습 내용:

- Redis에 Java 객체 저장
- JSON 직렬화 설정
- `RedisTemplate<String, Object>` 사용
- Spring Cache 추상화 적용
- `@Cacheable`, `@CachePut`, `@CacheEvict` 사용
- DB 조회 비용을 Redis 캐시로 줄이는 흐름 이해

---

## 2. 전체 구조

예제는 상품 정보를 기준으로 한다.

```text
상품 조회 요청
 → Redis 캐시 확인
 → 캐시에 있으면 Redis에서 반환
 → 캐시에 없으면 Service 로직 실행
 → 결과를 Redis에 저장
 → 다음 요청부터 Redis 캐시 사용
```

---

## 3. 의존성

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
    implementation 'org.springframework.boot:spring-boot-starter-cache'
    implementation 'org.springframework.boot:spring-boot-starter-validation'

    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

핵심 의존성:

| 의존성                           | 역할                     |
| -------------------------------- | ------------------------ |
| `spring-boot-starter-data-redis` | Redis 연동               |
| `spring-boot-starter-cache`      | Spring Cache 추상화 사용 |
| `spring-boot-starter-web`        | REST API 작성            |

---

## 4. application.yml

```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
      timeout: 3000ms

server:
  port: 8080
```

---

## 5. 패키지 구조

```text
src/main/java/com/example/rediscache
 ├─ RedisCacheApplication.java
 ├─ config
 │   └─ RedisConfig.java
 ├─ controller
 │   └─ ProductController.java
 ├─ service
 │   └─ ProductService.java
 ├─ repository
 │   └─ FakeProductRepository.java
 └─ dto
     ├─ ProductResponse.java
     └─ ProductUpdateRequest.java
```

---

## 6. 메인 클래스

`@EnableCaching`이 중요하다.

```java
package com.example.rediscache;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;

@EnableCaching
@SpringBootApplication
public class RedisCacheApplication {

    public static void main(String[] args) {
        SpringApplication.run(RedisCacheApplication.class, args);
    }
}
```

`@EnableCaching`을 붙여야 `@Cacheable`, `@CachePut`, `@CacheEvict`가 동작한다.

---

## 7. RedisConfig 작성

객체를 Redis에 저장하려면 직렬화 방식을 설정하는 것이 좋다.

```java
package com.example.rediscache.config;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;
import org.springframework.cache.CacheManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.time.Duration;

@Configuration
public class RedisConfig {

    @Bean
    public ObjectMapper redisObjectMapper() {
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.registerModule(new JavaTimeModule());
        objectMapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
        return objectMapper;
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate(
            RedisConnectionFactory connectionFactory,
            ObjectMapper redisObjectMapper
    ) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);

        StringRedisSerializer stringSerializer = new StringRedisSerializer();
        GenericJackson2JsonRedisSerializer jsonSerializer =
                new GenericJackson2JsonRedisSerializer(redisObjectMapper);

        template.setKeySerializer(stringSerializer);
        template.setHashKeySerializer(stringSerializer);
        template.setValueSerializer(jsonSerializer);
        template.setHashValueSerializer(jsonSerializer);

        template.afterPropertiesSet();
        return template;
    }

    @Bean
    public CacheManager cacheManager(
            RedisConnectionFactory connectionFactory,
            ObjectMapper redisObjectMapper
    ) {
        GenericJackson2JsonRedisSerializer jsonSerializer =
                new GenericJackson2JsonRedisSerializer(redisObjectMapper);

        RedisCacheConfiguration cacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofMinutes(10))
                .disableCachingNullValues()
                .serializeKeysWith(
                        RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer())
                )
                .serializeValuesWith(
                        RedisSerializationContext.SerializationPair.fromSerializer(jsonSerializer)
                );

        return RedisCacheManager.builder(connectionFactory)
                .cacheDefaults(cacheConfiguration)
                .build();
    }
}
```

---

## 8. DTO 작성

### 8.1 ProductResponse

```java
package com.example.rediscache.dto;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;

import java.io.Serializable;
import java.time.LocalDateTime;

@Getter
@NoArgsConstructor
@AllArgsConstructor
public class ProductResponse implements Serializable {

    private Long id;
    private String name;
    private int price;
    private LocalDateTime updatedAt;
}
```

`Serializable`은 필수는 아니지만 캐시 객체라는 의도를 명확히 하기 위해 붙였다.

---

### 8.2 ProductUpdateRequest

```java
package com.example.rediscache.dto;

import jakarta.validation.constraints.Min;
import jakarta.validation.constraints.NotBlank;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Getter
@NoArgsConstructor
public class ProductUpdateRequest {

    @NotBlank
    private String name;

    @Min(0)
    private int price;
}
```

---

## 9. Fake Repository 작성

실제 DB 대신 Map을 사용한다. 중요한 것은 Redis 캐시 흐름이다.

```java
package com.example.rediscache.repository;

import com.example.rediscache.dto.ProductResponse;
import org.springframework.stereotype.Repository;

import java.time.LocalDateTime;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

@Repository
public class FakeProductRepository {

    private final Map<Long, ProductResponse> store = new ConcurrentHashMap<>();

    public FakeProductRepository() {
        store.put(1L, new ProductResponse(1L, "Keyboard", 30000, LocalDateTime.now()));
        store.put(2L, new ProductResponse(2L, "Mouse", 15000, LocalDateTime.now()));
    }

    public ProductResponse findById(Long id) {
        sleepForDbDelay();
        return store.get(id);
    }

    public ProductResponse save(Long id, String name, int price) {
        ProductResponse product = new ProductResponse(id, name, price, LocalDateTime.now());
        store.put(id, product);
        return product;
    }

    public void delete(Long id) {
        store.remove(id);
    }

    private void sleepForDbDelay() {
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

`findById()`에 2초 지연을 넣었다. 캐시가 동작하면 첫 번째 조회는 느리고, 두 번째 조회는 빨라진다.

---

## 10. ProductService 작성

```java
package com.example.rediscache.service;

import com.example.rediscache.dto.ProductResponse;
import com.example.rediscache.dto.ProductUpdateRequest;
import com.example.rediscache.repository.FakeProductRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.CachePut;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class ProductService {

    private final FakeProductRepository productRepository;

    @Cacheable(cacheNames = "product", key = "#id")
    public ProductResponse findProduct(Long id) {
        ProductResponse product = productRepository.findById(id);

        if (product == null) {
            throw new IllegalArgumentException("상품을 찾을 수 없습니다. id=" + id);
        }

        return product;
    }

    @CachePut(cacheNames = "product", key = "#id")
    public ProductResponse updateProduct(Long id, ProductUpdateRequest request) {
        return productRepository.save(id, request.getName(), request.getPrice());
    }

    @CacheEvict(cacheNames = "product", key = "#id")
    public void deleteProduct(Long id) {
        productRepository.delete(id);
    }
}
```

---

## 11. Cache Annotation 설명

| 어노테이션    | 의미                                                                    |
| ------------- | ----------------------------------------------------------------------- |
| `@Cacheable`  | 캐시에 있으면 메서드 실행 안 하고 캐시 반환. 없으면 메서드 실행 후 저장 |
| `@CachePut`   | 메서드를 항상 실행하고 결과를 캐시에 갱신                               |
| `@CacheEvict` | 캐시 삭제                                                               |

### 11.1 `@Cacheable`

```java
@Cacheable(cacheNames = "product", key = "#id")
public ProductResponse findProduct(Long id) { ... }
```

Redis 키는 보통 다음과 유사하게 만들어진다.

```text
product::1
```

첫 조회:

```text
Redis 캐시 없음 → 메서드 실행 → DB 조회 → Redis 저장 → 응답
```

두 번째 조회:

```text
Redis 캐시 있음 → 메서드 실행 안 함 → Redis 값 응답
```

---

## 12. Controller 작성

```java
package com.example.rediscache.controller;

import com.example.rediscache.dto.ProductResponse;
import com.example.rediscache.dto.ProductUpdateRequest;
import com.example.rediscache.service.ProductService;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

import java.util.Map;

@RestController
@RequiredArgsConstructor
@RequestMapping("/api/products")
public class ProductController {

    private final ProductService productService;

    @GetMapping("/{id}")
    public ProductResponse findProduct(@PathVariable Long id) {
        return productService.findProduct(id);
    }

    @PutMapping("/{id}")
    public ProductResponse updateProduct(
            @PathVariable Long id,
            @Valid @RequestBody ProductUpdateRequest request
    ) {
        return productService.updateProduct(id, request);
    }

    @DeleteMapping("/{id}")
    public Map<String, Object> deleteProduct(@PathVariable Long id) {
        productService.deleteProduct(id);
        return Map.of("message", "deleted", "id", id);
    }
}
```

---

## 13. API 테스트

### 13.1 첫 번째 조회

```bash
curl http://localhost:8080/api/products/1
```

첫 번째 조회는 약 2초 걸린다.

### 13.2 두 번째 조회

```bash
curl http://localhost:8080/api/products/1
```

두 번째 조회는 Redis 캐시에서 반환되므로 빠르다.

### 13.3 Redis CLI 확인

```bash
docker exec -it redis-practice redis-cli
```

```bash
KEYS *
```

예상 키:

```bash
product::1
```

### 13.4 상품 수정

```bash
curl -X PUT http://localhost:8080/api/products/1 \
  -H "Content-Type: application/json" \
  -d '{"name":"Mechanical Keyboard", "price":80000}'
```

`@CachePut` 때문에 DB 역할의 Map이 수정되고 Redis 캐시도 새 값으로 갱신된다.

### 13.5 상품 삭제

```bash
curl -X DELETE http://localhost:8080/api/products/1
```

`@CacheEvict` 때문에 Redis 캐시도 삭제된다.

---

## 14. 직접 RedisTemplate으로 객체 저장하기

Spring Cache 말고 직접 RedisTemplate을 사용할 수도 있다.

### 14.1 ManualProductCacheService

```java
package com.example.rediscache.service;

import com.example.rediscache.dto.ProductResponse;
import lombok.RequiredArgsConstructor;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Service;

import java.time.Duration;

@Service
@RequiredArgsConstructor
public class ManualProductCacheService {

    private final RedisTemplate<String, Object> redisTemplate;

    public void saveProduct(ProductResponse product) {
        String key = "manual:product:" + product.getId();
        redisTemplate.opsForValue().set(key, product, Duration.ofMinutes(5));
    }

    public ProductResponse getProduct(Long id) {
        String key = "manual:product:" + id;
        Object value = redisTemplate.opsForValue().get(key);

        if (value == null) {
            return null;
        }

        return (ProductResponse) value;
    }

    public void deleteProduct(Long id) {
        String key = "manual:product:" + id;
        redisTemplate.delete(key);
    }
}
```

주의할 점:

- JSON 역직렬화 타입 문제가 생길 수 있다.
- 실무에서는 DTO 구조 변경에 주의해야 한다.
- 캐시 키 네이밍 규칙을 명확히 해야 한다.

---

## 15. 캐시 실무 패턴

### 15.1 Cache Aside Pattern

가장 흔한 패턴이다.

```text
1. 캐시 조회
2. 캐시에 없으면 DB 조회
3. DB 결과를 캐시에 저장
4. 응답 반환
```

`@Cacheable`이 이 패턴을 쉽게 만들어준다.

---

### 15.2 캐시 무효화

데이터 수정/삭제가 발생하면 캐시도 같이 지워야 한다.

```text
상품 수정
 → DB 수정
 → Redis 캐시 갱신 또는 삭제
```

그렇지 않으면 사용자가 오래된 데이터를 볼 수 있다.

---

### 15.3 TTL 설정

캐시는 영원히 저장하지 않는 것이 일반적이다.

```java
.entryTtl(Duration.ofMinutes(10))
```

TTL이 필요한 이유:

- 오래된 데이터 자동 제거
- 메모리 사용량 관리
- 장애 시 캐시 자연 회복

---

## 16. 자주 나는 오류

### 16.1 Redis 연결 실패

오류 예시:

```text
Unable to connect to Redis server
```

확인할 것:

```bash
docker ps
```

Redis 컨테이너가 떠 있는지 확인한다.

---

### 16.2 직렬화 오류

오류 예시:

```text
SerializationException
```

원인:

- 객체 직렬화 설정 누락
- LocalDateTime 직렬화 설정 누락
- 이전 직렬화 방식으로 저장된 값과 현재 설정 충돌

해결:

```bash
docker exec -it redis-practice redis-cli FLUSHALL
```

실습 환경에서만 사용한다. 운영에서 `FLUSHALL`은 절대 함부로 쓰면 안 된다.

---

## 17. 정리

| 주제                | 핵심                        |
| ------------------- | --------------------------- |
| RedisTemplate       | Redis를 직접 조작할 때 사용 |
| StringRedisTemplate | 문자열 Key-Value에 적합     |
| CacheManager        | Spring Cache와 Redis 연결   |
| `@Cacheable`        | 조회 캐싱                   |
| `@CachePut`         | 캐시 갱신                   |
| `@CacheEvict`       | 캐시 삭제                   |
| TTL                 | 캐시 자동 만료              |

Redis는 단순 저장소가 아니라 DB 부하를 줄이고 응답 속도를 높이는 핵심 캐시 계층으로 많이 사용된다.

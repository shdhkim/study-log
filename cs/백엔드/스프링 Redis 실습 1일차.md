# 01. Spring Boot Redis 기초 실습: 캐시, 문자열 저장, 세션성 데이터

## 1. 목표

이 파일에서는 Spring Boot에서 Redis를 연결하고 다음 기능을 실습한다.

- Redis 서버 연결
- `StringRedisTemplate` 사용
- 문자열 데이터 저장/조회/삭제
- TTL 만료 시간 설정
- 간단한 로그인 세션 토큰 저장 구조
- REST API로 Redis 동작 확인

Redis는 인메모리 Key-Value 저장소다. DB보다 빠르게 읽고 쓰기 때문에 캐시, 세션, 인증 토큰, 중복 요청 방지, 분산락 등에 자주 사용된다.

---

## 2. 실습 환경

예시 기준은 다음과 같다.

- Java 17
- Spring Boot 3.x
- Gradle
- Redis 7.x
- Lombok 사용

---

## 3. Redis 실행

### 3.1 Docker로 Redis 실행

```bash
docker run --name redis-practice -p 6379:6379 -d redis:7
```

정상 실행 확인:

```bash
docker ps
```

Redis CLI 접속:

```bash
docker exec -it redis-practice redis-cli
```

간단 테스트:

```bash
PING
```

응답:

```bash
PONG
```

---

## 4. 프로젝트 생성

Spring Initializr에서 다음 의존성을 선택한다.

- Spring Web
- Spring Data Redis
- Lombok
- Validation

또는 Gradle에 직접 추가한다.

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
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
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

## 5. application.yml 설정

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

설명:

| 설정                     | 의미                |
| ------------------------ | ------------------- |
| `spring.data.redis.host` | Redis 서버 주소     |
| `spring.data.redis.port` | Redis 포트          |
| `timeout`                | Redis 연결 타임아웃 |

---

## 6. 패키지 구조

```text
src/main/java/com/example/redisbasic
 ├─ RedisBasicApplication.java
 ├─ controller
 │   ├─ RedisStringController.java
 │   └─ LoginTokenController.java
 ├─ service
 │   ├─ RedisStringService.java
 │   └─ LoginTokenService.java
 └─ dto
     ├─ RedisSetRequest.java
     └─ LoginRequest.java
```

---

## 7. 메인 클래스

```java
package com.example.redisbasic;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class RedisBasicApplication {

    public static void main(String[] args) {
        SpringApplication.run(RedisBasicApplication.class, args);
    }
}
```

---

## 8. Redis 문자열 저장 실습

### 8.1 DTO 작성

```java
package com.example.redisbasic.dto;

import jakarta.validation.constraints.NotBlank;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Getter
@NoArgsConstructor
public class RedisSetRequest {

    @NotBlank
    private String key;

    @NotBlank
    private String value;

    private Long ttlSeconds;
}
```

`ttlSeconds`는 선택값이다. 값이 있으면 해당 초 이후 Redis 키가 자동 삭제된다.

---

### 8.2 Service 작성

```java
package com.example.redisbasic.service;

import lombok.RequiredArgsConstructor;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.stereotype.Service;

import java.time.Duration;

@Service
@RequiredArgsConstructor
public class RedisStringService {

    private final StringRedisTemplate stringRedisTemplate;

    public void set(String key, String value, Long ttlSeconds) {
        if (ttlSeconds == null) {
            stringRedisTemplate.opsForValue().set(key, value);
            return;
        }

        stringRedisTemplate.opsForValue().set(key, value, Duration.ofSeconds(ttlSeconds));
    }

    public String get(String key) {
        return stringRedisTemplate.opsForValue().get(key);
    }

    public Boolean delete(String key) {
        return stringRedisTemplate.delete(key);
    }

    public Long getExpire(String key) {
        return stringRedisTemplate.getExpire(key);
    }
}
```

핵심 API:

| 코드                            | 의미          |
| ------------------------------- | ------------- |
| `opsForValue().set(key, value)` | 문자열 저장   |
| `opsForValue().get(key)`        | 문자열 조회   |
| `delete(key)`                   | 키 삭제       |
| `getExpire(key)`                | 남은 TTL 조회 |

---

### 8.3 Controller 작성

```java
package com.example.redisbasic.controller;

import com.example.redisbasic.dto.RedisSetRequest;
import com.example.redisbasic.service.RedisStringService;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

import java.util.Map;

@RestController
@RequiredArgsConstructor
@RequestMapping("/api/redis/string")
public class RedisStringController {

    private final RedisStringService redisStringService;

    @PostMapping
    public Map<String, Object> set(@Valid @RequestBody RedisSetRequest request) {
        redisStringService.set(request.getKey(), request.getValue(), request.getTtlSeconds());

        return Map.of(
                "message", "saved",
                "key", request.getKey(),
                "value", request.getValue()
        );
    }

    @GetMapping("/{key}")
    public Map<String, Object> get(@PathVariable String key) {
        String value = redisStringService.get(key);
        Long ttl = redisStringService.getExpire(key);

        return Map.of(
                "key", key,
                "value", value,
                "ttl", ttl
        );
    }

    @DeleteMapping("/{key}")
    public Map<String, Object> delete(@PathVariable String key) {
        Boolean deleted = redisStringService.delete(key);

        return Map.of(
                "key", key,
                "deleted", deleted
        );
    }
}
```

---

## 9. API 테스트

### 9.1 TTL 없는 저장

```bash
curl -X POST http://localhost:8080/api/redis/string \
  -H "Content-Type: application/json" \
  -d '{"key":"user:1:name", "value":"kim"}'
```

### 9.2 조회

```bash
curl http://localhost:8080/api/redis/string/user:1:name
```

### 9.3 TTL 있는 저장

```bash
curl -X POST http://localhost:8080/api/redis/string \
  -H "Content-Type: application/json" \
  -d '{"key":"temp:code", "value":"123456", "ttlSeconds":30}'
```

30초 후 다시 조회하면 값이 사라진다.

### 9.4 삭제

```bash
curl -X DELETE http://localhost:8080/api/redis/string/user:1:name
```

---

## 10. 로그인 토큰 저장 실습

실제 서비스에서는 JWT나 세션 저장소를 더 정교하게 설계하지만, 여기서는 Redis TTL 기반 토큰 저장 흐름을 실습한다.

구조:

```text
login 요청
 → 토큰 생성
 → Redis에 token:{token} = userId 저장
 → TTL 30분 설정
 → 이후 토큰으로 사용자 확인
```

---

### 10.1 LoginRequest DTO

```java
package com.example.redisbasic.dto;

import jakarta.validation.constraints.NotBlank;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Getter
@NoArgsConstructor
public class LoginRequest {

    @NotBlank
    private String userId;

    @NotBlank
    private String password;
}
```

---

### 10.2 LoginTokenService

```java
package com.example.redisbasic.service;

import lombok.RequiredArgsConstructor;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.stereotype.Service;

import java.time.Duration;
import java.util.UUID;

@Service
@RequiredArgsConstructor
public class LoginTokenService {

    private static final Duration LOGIN_TOKEN_TTL = Duration.ofMinutes(30);
    private static final String TOKEN_KEY_PREFIX = "login:token:";

    private final StringRedisTemplate stringRedisTemplate;

    public String login(String userId, String password) {
        // 실습용이므로 비밀번호 검증은 단순 처리한다.
        // 실제 서비스에서는 DB 사용자 조회와 PasswordEncoder 검증이 필요하다.
        if (!"1234".equals(password)) {
            throw new IllegalArgumentException("비밀번호가 올바르지 않습니다.");
        }

        String token = UUID.randomUUID().toString();
        String redisKey = TOKEN_KEY_PREFIX + token;

        stringRedisTemplate.opsForValue().set(redisKey, userId, LOGIN_TOKEN_TTL);

        return token;
    }

    public String findUserIdByToken(String token) {
        String redisKey = TOKEN_KEY_PREFIX + token;
        return stringRedisTemplate.opsForValue().get(redisKey);
    }

    public void logout(String token) {
        String redisKey = TOKEN_KEY_PREFIX + token;
        stringRedisTemplate.delete(redisKey);
    }

    public void extendToken(String token) {
        String redisKey = TOKEN_KEY_PREFIX + token;
        Boolean exists = stringRedisTemplate.hasKey(redisKey);

        if (Boolean.FALSE.equals(exists)) {
            throw new IllegalArgumentException("유효하지 않은 토큰입니다.");
        }

        stringRedisTemplate.expire(redisKey, LOGIN_TOKEN_TTL);
    }
}
```

---

### 10.3 LoginTokenController

```java
package com.example.redisbasic.controller;

import com.example.redisbasic.dto.LoginRequest;
import com.example.redisbasic.service.LoginTokenService;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

import java.util.Map;

@RestController
@RequiredArgsConstructor
@RequestMapping("/api/login")
public class LoginTokenController {

    private final LoginTokenService loginTokenService;

    @PostMapping
    public Map<String, Object> login(@Valid @RequestBody LoginRequest request) {
        String token = loginTokenService.login(request.getUserId(), request.getPassword());

        return Map.of(
                "message", "login success",
                "token", token
        );
    }

    @GetMapping("/me")
    public Map<String, Object> me(@RequestHeader("Authorization") String authorization) {
        String token = removeBearerPrefix(authorization);
        String userId = loginTokenService.findUserIdByToken(token);

        if (userId == null) {
            throw new IllegalArgumentException("로그인이 필요합니다.");
        }

        return Map.of(
                "userId", userId
        );
    }

    @PostMapping("/extend")
    public Map<String, Object> extend(@RequestHeader("Authorization") String authorization) {
        String token = removeBearerPrefix(authorization);
        loginTokenService.extendToken(token);

        return Map.of("message", "token extended");
    }

    @PostMapping("/logout")
    public Map<String, Object> logout(@RequestHeader("Authorization") String authorization) {
        String token = removeBearerPrefix(authorization);
        loginTokenService.logout(token);

        return Map.of("message", "logout success");
    }

    private String removeBearerPrefix(String authorization) {
        if (authorization == null || !authorization.startsWith("Bearer ")) {
            throw new IllegalArgumentException("Authorization 헤더 형식이 올바르지 않습니다.");
        }
        return authorization.substring(7);
    }
}
```

---

## 11. 로그인 API 테스트

### 11.1 로그인

```bash
curl -X POST http://localhost:8080/api/login \
  -H "Content-Type: application/json" \
  -d '{"userId":"user01", "password":"1234"}'
```

응답 예시:

```json
{
  "message": "login success",
  "token": "b2a6717e-7d54-4b46-9c2a-3d2dfc1ed7aa"
}
```

### 11.2 내 정보 조회

```bash
curl http://localhost:8080/api/login/me \
  -H "Authorization: Bearer 발급받은토큰"
```

### 11.3 토큰 만료 연장

```bash
curl -X POST http://localhost:8080/api/login/extend \
  -H "Authorization: Bearer 발급받은토큰"
```

### 11.4 로그아웃

```bash
curl -X POST http://localhost:8080/api/login/logout \
  -H "Authorization: Bearer 발급받은토큰"
```

---

## 12. Redis CLI로 확인

```bash
docker exec -it redis-practice redis-cli
```

키 목록 확인:

```bash
KEYS *
```

값 확인:

```bash
GET login:token:토큰값
```

TTL 확인:

```bash
TTL login:token:토큰값
```

삭제 확인:

```bash
DEL login:token:토큰값
```

---

## 13. 실무 관점 정리

| 용도           | Redis 사용 이유                        |
| -------------- | -------------------------------------- |
| 캐시           | DB 조회 부하 감소                      |
| 로그인 세션    | TTL 기반 자동 만료                     |
| 인증 코드      | 짧은 시간 동안만 값 보관               |
| 중복 요청 방지 | 특정 키를 짧은 시간 동안 잠금처럼 사용 |
| 분산락         | 여러 서버에서 같은 자원 동시 처리 방지 |

---

## 14. 주의사항

### 14.1 `KEYS *`는 운영에서 조심

`KEYS *`는 전체 키를 스캔하므로 운영 Redis에서 성능 문제가 생길 수 있다. 운영에서는 `SCAN`을 사용하는 것이 좋다.

### 14.2 Redis는 메모리 기반

Redis는 빠르지만 메모리를 사용한다. 너무 큰 데이터를 넣거나 TTL 없이 무한히 저장하면 메모리 부족이 발생할 수 있다.

### 14.3 중요한 데이터의 원본 저장소로 쓰지 않기

Redis는 캐시나 임시 저장소로 많이 쓴다. 반드시 보존해야 하는 핵심 데이터는 RDBMS 같은 영속 저장소에 저장하는 것이 일반적이다.

---

## 15. 다음 실습 방향

다음 단계에서는 Redis에 객체 JSON을 저장하고, Spring Cache 추상화를 사용하여 DB 조회 결과를 캐싱하는 구조를 실습할 수 있다.

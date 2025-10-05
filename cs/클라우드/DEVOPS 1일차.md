
# CI/CD와 Jenkins 상세 정리

---

# 1. CI/CD란?

## 1.1 CI (Continuous Integration, 지속적 통합)

- 개발자가 작성한 소스를 하나의 저장소에 자주 통합하는 문화 및 방법론
- 지속적으로 소스코드를 통합하고, 자동으로 버그 확인, 빌드, 테스트를 진행
- 보통 개발자가 Git에 커밋하면 자동으로 빌드와 테스트가 실행됨

## 1.2 CD (Continuous Delivery / Deployment, 지속적 제공 / 지속적 배포)

- **Continuous Delivery**: 자동으로 파이프라인을 구성하고, 수동으로 서비스 업데이트 가능
- **Continuous Deployment**: 파이프라인을 통과하면 자동으로 운영 서버에 배포까지 진행
- 여기서는 일반적으로 "지속적 배포" 의미로 사용

## 1.3 CI/CD 흐름 요약

```
[개발자가 Git에 커밋]
    ↓
[CI] 자동 빌드 및 자동 테스트
    ↓
[CD] 자동 파이프라인 구성 및 운영 서버 배포
    ↓
[운영 서버 모니터링 및 롤백 준비]
```

### CI/CD 흐름 그림 요약

```
[개발자]
    ↓ (코드 커밋)
[Git 저장소]
    ↓ (웹훅 트리거)
[Jenkins 서버]
    - 빌드
    - 테스트
    - 패키징
    ↓
[Docker/Kubernetes]
    ↓
[운영 서버에 배포]
    ↓
[모니터링 및 자동 롤백]
```

---

# 2. Jenkins 상세 설명

## 2.1 Jenkins란?

- 오픈소스 기반의 자동화 서버로, 소프트웨어 빌드, 테스트, 배포를 지원하는 툴
- 다양한 플러그인을 통해 확장 가능하며, CI/CD 파이프라인 구축에 가장 널리 사용됨
- Java로 개발되었으며, 다양한 플랫폼과 연동 가능

## 2.2 Jenkins 주요 기능

| 키워드 | 설명 |
|:---|:---|
| 테스트 자동 실행 | 코드가 변경되면 자동으로 테스트 수행 |
| 빌드 자동 실행 | 개발 완료 후 자동으로 컴파일 및 패키징 |
| 자동 배포 | Docker, Kubernetes, AWS 등에 자동 배포 가능 |
| 파이프라인 구성 | 복잡한 작업 흐름을 Jenkinsfile로 정의 가능 |

## 2.3 Jenkins 구성 흐름

```
[소스 코드 저장소 (GitHub, GitLab 등)]
    ↓ (Webhook 또는 주기적 체크)
[Jenkins 서버]
    - 작업(Pipeline) 실행:
        - 빌드
        - 테스트
        - Docker 이미지 생성
        - Kubernetes에 배포
    ↓
[운영 서버 (Production Environment)]
```

---

# 3. Jenkins 실습 예시 및 CI/CD 흐름

## 3.1 Jenkins 실습 전체 플로우

```
[1] 개발자가 GitHub에 코드 Push
      ↓
[2] GitHub Webhook이 Jenkins 서버 호출
      ↓
[3] Jenkins가 CI 파이프라인 실행
      - 빌드 (gradle build)
      - 테스트 (gradle test)
      - Docker 이미지 빌드
      - Docker Hub에 푸시
      ↓
[4] Jenkins가 CD 파이프라인 실행
      - Kubernetes에 새로운 버전 배포 (kubectl apply)
      ↓
[5] 운영 서버 업데이트 완료
      ↓
[6] Prometheus + Grafana로 모니터링
      ↓
[7] 문제 발생 시 자동 롤백 (Argo Rollouts, Helm rollback 등)
```

## 3.2 Jenkins Pipeline 실제 예시

```groovy
pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'myapp:${BUILD_NUMBER}'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/myrepo/myapp.git'
            }
        }
        stage('Build') {
            steps {
                sh './gradlew build'
            }
        }
        stage('Test') {
            steps {
                sh './gradlew test'
            }
        }
        stage('Docker Build and Push') {
            steps {
                sh 'docker build -t mydockerhub/myapp:$DOCKER_IMAGE .'
                sh 'docker push mydockerhub/myapp:$DOCKER_IMAGE'
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl set image deployment/myapp myapp=mydockerhub/myapp:$DOCKER_IMAGE'
            }
        }
    }
}
```

---

# 4. CI/CD 전체 흐름 그림 (최종 요약)

```
[개발자]
    ↓ (코드 커밋)
[Git 저장소 (GitHub, GitLab)]
    ↓ (Webhook Trigger)
[Jenkins]
    - Checkout 소스
    - 빌드 (Gradle, Maven)
    - 테스트 (JUnit, Mockito)
    - Docker 이미지 빌드
    - Docker 이미지 푸시
    - Kubernetes 배포
    ↓
[운영 서버]
    ↓
[모니터링 시스템 (Prometheus, Grafana)]
    ↓
[문제 발생 시 롤백 자동화]
```

---

#  최종 정리

 Jenkins는 CI/CD 자동화 파이프라인의 중심 역할을 한다.  
 Jenkinsfile을 통해 빌드, 테스트, 배포를 코드로 관리할 수 있다.  
 Docker, Kubernetes, Prometheus와 연계하여 강력한 DevOps 환경을 구성할 수 있다.

---




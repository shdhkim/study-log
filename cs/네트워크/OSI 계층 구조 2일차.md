# 네이버 접속으로 이해하는 OSI 7계층, Web Server, WAS, React, Nginx, Reverse Proxy, Load Balancer

## 1. 네이버 접속 예시

브라우저에서 다음 주소를 입력한다고 가정한다.

```text
https://www.naver.com
```

사용자 입장에서는 주소를 입력하고 엔터만 누른 것처럼 보인다.

하지만 내부적으로는 여러 계층을 거쳐 데이터가 만들어지고, 포장되고, 전송되고, 다시 풀리면서 네이버 화면이 나타난다.

전체 흐름은 다음과 같다.

```text
브라우저
↓
DNS 조회
↓
네이버 서버 IP 확인
↓
TCP 연결
↓
TLS 암호화 연결
↓
HTTP 요청
↓
Nginx 또는 Load Balancer
↓
Web Server / WAS
↓
DB 또는 내부 서비스
↓
HTTP 응답
↓
브라우저 화면 출력
```

---

## 2. OSI 7계층 전체 흐름

OSI 7계층은 네트워크 통신을 역할별로 나눈 모델이다.

송신자는 위에서 아래로 내려가며 데이터를 포장한다.

```text
7 → 6 → 5 → 4 → 3 → 2 → 1
```

수신자는 아래에서 위로 올라가며 포장을 푼다.

```text
1 → 2 → 3 → 4 → 5 → 6 → 7
```

쉽게 말하면 다음과 같다.

```text
보내는 쪽 = 포장
받는 쪽 = 포장 해제
```

---

# 3. 7계층 Application Layer

## 역할

Application Layer는 사용자가 직접 사용하는 서비스 계층이다.

예시

```text
브라우저
카카오톡
메일
게임
파일 전송 프로그램
```

프로토콜 예시

```text
HTTP
HTTPS
FTP
SMTP
DNS
```

네이버 접속에서는 브라우저가 HTTP 요청을 만든다.

```http
GET / HTTP/1.1
Host: www.naver.com
```

의미는 다음과 같다.

```text
www.naver.com 서버야,
메인 페이지를 보내줘.
```

이 계층에서 만들어지는 것은 사용자가 원하는 요청 내용이다.

---

# 4. 6계층 Presentation Layer

## 역할

Presentation Layer는 데이터의 표현 방식을 처리한다.

대표 역할

```text
암호화
복호화
압축
인코딩
디코딩
```

HTTPS에서는 TLS가 이 역할과 관련된다.

HTTP 요청은 원래 사람이 읽을 수 있는 형태이다.

```http
GET /login HTTP/1.1
id=kim
pw=1234
```

하지만 HTTPS에서는 이 데이터를 TLS로 암호화한다.

```text
HTTP 데이터
↓
TLS 암호화
↓
암호문
```

암호문 예시

```text
9A8DFA91KDL39AKD...
```

중간에서 누가 패킷을 가로채도 내용을 읽기 어렵다.

즉 HTTP는 엽서처럼 내용이 보이고, HTTPS는 봉투에 넣고 잠근 편지처럼 내용이 보이지 않는다.

---

# 5. 5계층 Session Layer

## 역할

Session Layer는 통신 세션을 관리한다.

세션이란 대화의 흐름이다.

예를 들어 브라우저와 네이버 서버가 통신 중이라면 다음 상태를 유지해야 한다.

```text
이 사용자는 네이버 서버와 통신 중이다.
이 연결은 아직 유지 중이다.
이 요청과 응답은 같은 흐름에 속한다.
```

현대 인터넷에서는 OSI 5계층 기능이 TCP, TLS, 애플리케이션 로직에 섞여 있는 경우가 많다.

예시

```text
로그인 상태 유지
연결 생성
연결 유지
연결 종료
```

---

# 6. 4계층 Transport Layer

## 역할

Transport Layer는 프로세스 간 통신을 담당한다.

대표 프로토콜

```text
TCP
UDP
```

HTTPS는 보통 TCP를 사용한다.

TCP가 하는 일

```text
데이터 분할
순서 번호 부여
ACK 확인
재전송
흐름 제어
혼잡 제어
Port 번호 사용
```

예를 들어 내 PC에서 네이버에 접속하면 다음과 같은 포트가 붙는다.

```text
출발 Port : 53122
도착 Port : 443
```

443은 HTTPS 서버의 기본 포트이다.

이 계층에서 만들어진 단위를 보통 Segment라고 한다.

```text
TCP Header + 데이터 = TCP Segment
```

TCP Header에는 다음 정보가 들어갈 수 있다.

```text
출발 포트
목적지 포트
순서 번호
ACK 번호
SYN
ACK
FIN
Window Size
Checksum
```

---

## TCP 3-Way Handshake

HTTP 요청을 보내기 전에 TCP 연결을 먼저 만든다.

```text
1. 클라이언트 → 서버 : SYN
2. 서버 → 클라이언트 : SYN + ACK
3. 클라이언트 → 서버 : ACK
```

이 과정을 통해 양쪽이 통신 가능한 상태인지 확인한다.

그 후 실제 데이터가 전송된다.

---

# 7. 3계층 Network Layer

## 역할

Network Layer는 목적지 컴퓨터까지 데이터를 보내기 위한 주소와 경로를 담당한다.

대표 프로토콜

```text
IP
ICMP
```

IP 주소를 사용한다.

예시

```text
출발 IP : 내 PC IP
목적지 IP : 네이버 서버 IP
```

이 계층에서 만들어진 단위를 Packet이라고 한다.

```text
IP Header + TCP Segment = IP Packet
```

라우터는 IP 주소를 보고 다음 경로를 결정한다.

```text
내 PC
↓
공유기
↓
통신사 라우터
↓
여러 라우터
↓
네이버 서버 측 네트워크
```

IP Header에는 다음 정보가 들어갈 수 있다.

```text
출발 IP
목적지 IP
TTL
프로토콜 번호
패킷 길이
Header Checksum
```

---

# 8. 2계층 Data Link Layer

## 역할

Data Link Layer는 같은 네트워크 구간 안에서 데이터를 전달한다.

대표 기술

```text
Ethernet
Wi-Fi
HDLC
PPP
```

이 계층에서는 MAC 주소를 사용한다.

예시

```text
출발 MAC : 내 PC MAC
목적지 MAC : 공유기 MAC
```

중요한 점은 목적지 IP와 목적지 MAC이 항상 같은 대상은 아니라는 것이다.

네이버 접속 시

```text
목적지 IP : 네이버 서버 IP
목적지 MAC : 공유기 MAC
```

왜냐하면 내 PC는 네이버 서버와 직접 연결되어 있지 않기 때문이다.

일단 같은 네트워크 안에 있는 공유기에게 보내고, 그 다음부터 라우터들이 목적지까지 전달한다.

이 계층에서 만들어진 단위를 Frame이라고 한다.

```text
Ethernet Header + IP Packet + FCS = Frame
```

FCS는 오류 검사용이다.

Ethernet Frame 예시

```text
목적지 MAC
출발지 MAC
Type
Data(IP Packet)
FCS
```

---

# 9. 1계층 Physical Layer

## 역할

Physical Layer는 비트를 실제 신호로 바꿔 전송한다.

전송되는 값은 결국 0과 1이다.

```text
0101010101010101
```

매체에 따라 신호 형태가 달라진다.

```text
LAN 케이블 → 전기 신호
광케이블 → 빛
Wi-Fi → 전파
```

즉 1계층은 실제 물리적인 전송을 담당한다.

---

# 10. 송신 과정 전체

브라우저가 HTTP 요청을 만든다.

```text
HTTP 데이터
```

6계층에서 TLS 암호화가 적용된다.

```text
암호화된 HTTP 데이터
```

4계층에서 TCP Header가 붙는다.

```text
TCP Header + 암호화된 데이터
```

3계층에서 IP Header가 붙는다.

```text
IP Header + TCP Header + 암호화된 데이터
```

2계층에서 Ethernet Header와 FCS가 붙는다.

```text
Ethernet Header + IP Header + TCP Header + 암호화된 데이터 + FCS
```

1계층에서 비트 신호로 변환된다.

```text
0101010101010101
```

---

# 11. 수신 과정 전체

서버는 반대로 처리한다.

```text
전기/빛/전파 신호 수신
↓
비트 복원
↓
Ethernet Frame 확인
↓
IP Packet 확인
↓
TCP Segment 확인
↓
TLS 복호화
↓
HTTP 요청 확인
```

최종적으로 서버는 다음 요청을 읽는다.

```http
GET / HTTP/1.1
Host: www.naver.com
```

그리고 응답을 만든다.

```http
HTTP/1.1 200 OK
Content-Type: text/html

<html>
...
</html>
```

응답도 다시 같은 과정을 거쳐 사용자 브라우저로 돌아간다.

---

# 12. Web Server란?

Web Server는 이미 만들어진 정적 파일을 그대로 제공하는 서버이다.

대표 프로그램

```text
Nginx
Apache
```

정적 파일 예시

```text
index.html
main.js
style.css
logo.png
font.woff
```

브라우저가 다음 요청을 보낸다.

```http
GET /logo.png
```

Nginx는 디스크에서 logo.png를 찾아 그대로 보낸다.

```text
브라우저
↓
Nginx
↓
logo.png 읽기
↓
브라우저
```

Java 코드 실행도 없고 DB 조회도 없다.

그냥 파일을 읽어서 전달한다.

---

# 13. WAS란?

WAS는 Web Application Server이다.

대표 예시

```text
Spring Boot
Tomcat
JBoss
Node.js 서버
FastAPI
```

WAS는 요청을 받을 때마다 프로그램을 실행한다.

예를 들어 다음 요청이 온다.

```http
GET /api/member/1
```

Spring Boot에서는 컨트롤러가 실행된다.

```java
@GetMapping("/api/member/{id}")
public MemberDto getMember(@PathVariable Long id) {
    return memberService.findById(id);
}
```

이 과정에서 DB 조회가 발생할 수 있다.

```sql
SELECT *
FROM MEMBER
WHERE ID = 1;
```

그리고 JSON 응답을 만든다.

```json
{
  "id": 1,
  "name": "Kim"
}
```

즉 WAS는 정적 파일만 주는 것이 아니라 프로그램을 실행해서 응답을 만든다.

---

# 14. Web Server와 WAS 차이

| 구분           | Web Server             | WAS                    |
| -------------- | ---------------------- | ---------------------- |
| 대표           | Nginx, Apache          | Spring Boot, Tomcat    |
| 주요 역할      | 정적 파일 제공         | 프로그램 실행          |
| DB 조회        | 보통 안 함             | 가능                   |
| Java 코드 실행 | 안 함                  | 함                     |
| 예시 요청      | /index.html, /logo.png | /api/member, /login    |
| 응답 방식      | 파일 그대로 전달       | 로직 실행 후 응답 생성 |

---

# 15. React는 어디서 실행될까?

React는 서버에서 실행되는 것이 아니라 브라우저에서 실행된다.

React 개발 코드는 다음과 같다.

```jsx
function App() {
  return <h1>Hello</h1>;
}
```

배포할 때는 빌드한다.

```bash
npm run build
```

그러면 다음과 같은 정적 파일이 생성된다.

```text
index.html
assets/main.js
assets/style.css
```

Nginx는 이 파일을 브라우저에게 전달한다.

브라우저는 main.js를 다운로드하고 실행한다.

```text
Nginx
↓
main.js 전달
↓
브라우저
↓
JavaScript 엔진이 main.js 실행
↓
React 실행
```

즉 React는 동적인 화면을 만들지만, 배포되는 형태는 정적 파일이다.

---

# 16. React 실행 과정 상세

1. 사용자가 사이트 접속

```text
https://example.com
```

2. 브라우저가 요청

```http
GET /
```

3. Nginx가 index.html 반환

```html
<div id="root"></div>
<script src="/assets/main.js"></script>
```

4. 브라우저가 script 태그를 발견

```http
GET /assets/main.js
```

5. Nginx가 main.js 반환

6. 브라우저 JavaScript 엔진이 main.js 실행

7. main.js 내부의 React 코드 실행

```javascript
ReactDOM.createRoot(document.getElementById("root")).render(<App />);
```

8. 화면에 React UI 출력

---

# 17. Reverse Proxy란?

Reverse Proxy는 사용자의 요청을 대신 받아 적절한 서버로 전달하는 역할이다.

예를 들어 다음과 같이 나눌 수 있다.

```text
/        → React 정적 파일
/api     → Spring Boot
/ai      → FastAPI
/images  → 이미지 서버
```

사용자는 항상 같은 도메인으로 요청한다.

```text
https://example.com
```

하지만 Nginx는 경로에 따라 다른 서버로 보낸다.

```text
사용자
↓
Nginx
├── /      → React 정적 파일
├── /api   → Spring Boot
└── /ai    → FastAPI
```

이것이 Reverse Proxy이다.

---

# 18. Load Balancer란?

Load Balancer는 같은 서비스를 여러 서버에 나눠주는 역할이다.

예를 들어 Spring Boot 서버가 3대 있다고 하자.

```text
Spring1
Spring2
Spring3
```

Load Balancer는 요청을 분산한다.

```text
사용자1 → Spring1
사용자2 → Spring2
사용자3 → Spring3
사용자4 → Spring1
```

대표 알고리즘

```text
Round Robin
Least Connections
IP Hash
Weighted Round Robin
```

---

# 19. Reverse Proxy와 Load Balancer 차이

Reverse Proxy는 서비스별로 보낸다.

```text
/api → Spring
/ai → FastAPI
```

Load Balancer는 같은 서비스를 여러 서버로 분산한다.

```text
/api → Spring1
/api → Spring2
/api → Spring3
```

즉 기준이 다르다.

```text
Reverse Proxy = 어느 서비스로 보낼까?
Load Balancer = 같은 서비스 중 어느 서버로 보낼까?
```

---

# 20. Nginx는 무엇인가?

Nginx는 실행되는 프로그램이다.

Nginx는 다음 역할을 모두 할 수 있다.

```text
Web Server
Reverse Proxy
Load Balancer
TLS 종료
압축
캐싱
```

예시 구조

```text
브라우저
↓
Nginx
├── /       → index.html, main.js 제공
├── /api    → Spring Boot로 전달
└── /image  → 이미지 파일 제공
```

Spring 서버가 여러 대라면 Nginx가 로드 밸런싱도 할 수 있다.

```text
Nginx
↓
Spring1
Spring2
Spring3
```

---

# 21. 전체 구조 예시

```text
사용자 브라우저
↓
DNS 조회
↓
IP 확인
↓
TCP 연결
↓
TLS 연결
↓
HTTP 요청
↓
Nginx
├── 정적 파일 요청 → index.html, main.js, style.css
└── API 요청 → Spring Boot
                  ↓
                 DB
```

React 화면 로딩 과정

```text
브라우저
↓
GET /
↓
Nginx
↓
index.html 반환
↓
브라우저
↓
GET /assets/main.js
↓
Nginx
↓
main.js 반환
↓
브라우저에서 React 실행
```

API 요청 과정

```text
React
↓
axios.get("/api/member")
↓
브라우저
↓
Nginx
↓
Spring Boot
↓
DB
↓
JSON 응답
↓
React 화면 갱신
```

---

# 22. Wireshark에서는 무엇을 볼 수 있을까?

Wireshark는 패킷 분석 도구이다.

확인 가능한 것

```text
Ethernet Frame
MAC 주소
IP 주소
TCP Port
TCP 3-Way Handshake
TLS Handshake
DNS 요청
HTTP 요청
ACK
SYN
FIN
재전송
패킷 크기
응답 시간
```

HTTP라면 요청 내용이 보인다.

```http
GET /api/member HTTP/1.1
```

HTTPS라면 내용은 암호화되어 보이지 않고 다음처럼 보인다.

```text
TLS Application Data
```

---

# 23. 핵심 요약

## OSI 7계층

```text
7 Application  : HTTP 요청 생성
6 Presentation : TLS 암호화
5 Session      : 연결 상태 관리
4 Transport    : TCP, Port, 재전송
3 Network      : IP, 라우팅
2 Data Link    : MAC, Frame
1 Physical     : 전기, 빛, 전파
```

## Web Server

```text
정적 파일 제공
예: Nginx, Apache
```

## WAS

```text
프로그램 실행
DB 조회
JSON 응답 생성
예: Spring Boot, Tomcat
```

## React

```text
빌드 후 HTML, CSS, JS 정적 파일이 됨
브라우저에서 실행됨
```

## Reverse Proxy

```text
/api → Spring
/ai → FastAPI
```

서비스별로 전달한다.

## Load Balancer

```text
Spring1
Spring2
Spring3
```

같은 서비스를 여러 서버에 분산한다.

## Nginx

```text
정적 파일 제공
Reverse Proxy
Load Balancer
TLS 처리
캐싱
압축
```

---

# 24. 한 줄 정리

브라우저가 네이버에 접속하면 OSI 7계층을 따라 데이터가 포장되어 전송되고, Nginx 같은 Web Server는 정적 파일을 제공하거나 요청을 WAS로 전달하며, WAS는 프로그램을 실행해 동적 응답을 만든다. React는 Nginx가 전달한 JavaScript 파일을 브라우저가 실행하면서 동작하고, API 요청은 Nginx를 거쳐 Spring Boot 같은 WAS로 전달된다.

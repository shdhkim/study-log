# OSI 7계층 전송 과정

## 전체 흐름

송신자는 위에서 아래로 내려간다.

```text
7 → 6 → 5 → 4 → 3 → 2 → 1
```

수신자는 아래에서 위로 올라간다.

```text
1 → 2 → 3 → 4 → 5 → 6 → 7
```

쉽게 말하면

```text
보내는 쪽 = 포장
받는 쪽 = 포장 해제
```

---

# 예시

브라우저에서 접속한다고 하자.

```text
https://www.naver.com
```

---

# 7계층 Application

사용자가 사용하는 프로그램 계층이다.

예시

```text
Chrome
Edge
카카오톡
게임
```

브라우저가 HTTP 요청을 만든다.

```http
GET / HTTP/1.1
Host: www.naver.com
```

의미는

```text
네이버 메인 페이지 주세요.
```

이다.

---

# 6계층 Presentation

데이터 표현 방식을 처리한다.

HTTPS라면 TLS 암호화를 한다.

```text
HTTP 데이터

↓

TLS 암호화

↓

암호문
```

중간에서 누가 봐도 내용을 읽기 어렵다.

---

# 5계층 Session

통신 연결 상태를 관리한다.

쉽게 말하면

```text
내 브라우저와 네이버 서버가 대화 중이다.
```

라는 상태를 관리한다.

현대 인터넷에서는 TCP/TLS가 이 역할을 많이 담당한다.

---

# 4계층 Transport

TCP 또는 UDP가 동작한다.

HTTPS는 보통 TCP를 사용한다.

TCP가 하는 일

```text
데이터 분할
순서번호 부여
ACK 확인
재전송
Port 추가
```

예시

```text
출발 Port : 53122
도착 Port : 443
```

443은 HTTPS 서버 포트이다.

이 계층에서 만들어진 단위를 보통

```text
Segment
```

라고 한다.

형태는 대략 이렇다.

```text
TCP Header + 데이터
```

---

# 3계층 Network

IP가 동작한다.

IP 주소를 붙인다.

예시

```text
출발 IP : 내 PC IP
목적지 IP : 네이버 서버 IP
```

라우터는 목적지 IP를 보고 길을 찾는다.

이 계층에서 만들어진 단위를

```text
Packet
```

이라고 한다.

형태는 대략 이렇다.

```text
IP Header + TCP Segment
```

---

# 2계층 Data Link

Ethernet 또는 Wi-Fi가 동작한다.

MAC 주소를 붙인다.

예시

```text
출발 MAC : 내 PC MAC
목적지 MAC : 공유기 MAC
```

주의할 점

```text
목적지 IP는 네이버 서버 IP
목적지 MAC은 공유기 MAC
```

왜냐하면 내 PC는 네이버 서버와 직접 연결된 것이 아니라,
일단 공유기에게 보내야 하기 때문이다.

이 계층에서 만들어진 단위를

```text
Frame
```

이라고 한다.

형태는 대략 이렇다.

```text
Ethernet Header + IP Packet + FCS
```

FCS는 오류검사용이다.

---

# 1계층 Physical

Frame을 실제 신호로 바꾼다.

```text
0101010101010101
```

전송 방식

```text
LAN 케이블 → 전기 신호
광케이블 → 빛
Wi-Fi → 전파
```

---

# 중간 장비

## 스위치

스위치는 2계층 장비이다.

MAC 주소를 보고 전달한다.

```text
목적지 MAC 확인

↓

해당 포트로 전송
```

---

## 라우터

라우터는 3계층 장비이다.

IP 주소를 보고 전달한다.

```text
목적지 IP 확인

↓

다음 네트워크로 전송
```

라우터를 지날 때마다 MAC 주소는 바뀔 수 있다.

하지만 목적지 IP는 보통 그대로 유지된다.

---

# 받는 쪽 과정

서버는 반대로 뜯는다.

```text
1계층: 전기/전파/빛 신호 수신

↓

2계층: Ethernet/Wi-Fi Frame 확인
        MAC 주소 확인
        FCS 오류검사

↓

3계층: IP Packet 확인
        목적지 IP가 자기 IP인지 확인

↓

4계층: TCP Segment 확인
        Port 443 확인
        순서 맞추기
        ACK 처리

↓

6계층: TLS 복호화

↓

7계층: HTTP 요청 해석
```

최종적으로 서버는 이것을 읽는다.

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

응답도 다시 같은 방식으로 내려간다.

```text
7 → 6 → 5 → 4 → 3 → 2 → 1
```

사용자 PC는 다시 역순으로 받는다.

```text
1 → 2 → 3 → 4 → 5 → 6 → 7
```

---

# 계층별 포장 구조

송신할 때

```text
HTTP 데이터
```

↓

```text
TCP Header + HTTP 데이터
```

↓

```text
IP Header + TCP Header + HTTP 데이터
```

↓

```text
Ethernet Header + IP Header + TCP Header + HTTP 데이터 + FCS
```

↓

```text
0101010101010101
```

---

# 수신할 때

```text
0101010101010101
```

↓

```text
Ethernet Header 확인 후 제거
```

↓

```text
IP Header 확인 후 제거
```

↓

```text
TCP Header 확인 후 제거
```

↓

```text
TLS 복호화
```

↓

```text
HTTP 데이터 읽기
```

---

# 계층별 핵심 정리

| 계층 | 이름         | 핵심 역할              | 대표 예시             |
| ---- | ------------ | ---------------------- | --------------------- |
| 7    | Application  | 사용자 서비스          | HTTP                  |
| 6    | Presentation | 암호화, 인코딩         | TLS                   |
| 5    | Session      | 연결 관리              | 세션                  |
| 4    | Transport    | 포트, 재전송, 순서보장 | TCP, UDP              |
| 3    | Network      | IP 주소, 라우팅        | IP                    |
| 2    | Data Link    | MAC 주소, 프레임       | Ethernet, Wi-Fi, HDLC |
| 1    | Physical     | 실제 신호 전송         | 전기, 빛, 전파        |

---

# 한 줄 정리

OSI 7계층은 데이터를 보낼 때 각 계층이 Header를 붙여 포장하고,
받을 때는 각 계층이 Header를 제거하면서 원래 데이터를 복원하는 구조이다.

# TIL: Application Layer — Web and HTTP (기본 개념, Overview, Connections)

> **범위:** Chapter 2 — Web and HTTP 전반부 (슬라이드 2-18 ~ 2-25)  

---

## 1. Web 기본 개념

### Web page의 구조

Web page는 **object들의 모음**이다.

```
Web page = base HTML file + 여러 개의 referenced objects
```

- object: HTML 파일, JPEG 이미지, 오디오 파일 등 하나의 파일 단위
- base HTML 파일이 뼈대 역할을 하고, 그 안에 다른 object들을 참조(reference)
- 각 object는 서로 다른 서버에 저장될 수 있음

### URL (Uniform Resource Locator)

모든 object는 URL로 식별된다.

```
www.someschool.edu/someDept/pic.gif
├── host name ──┤├── path name ──┤
```

| 부분 | 역할 |
|------|------|
| host name | 어떤 서버인지 |
| path name | 그 서버 안에서 어디에 있는지 |

- IP + port가 "어떤 컴퓨터의 어떤 프로세스"를 찾는 것이라면, URL은 **"어떤 서버의 어떤 파일"**을 찾는 것

---

## 2. HTTP Overview

### HTTP란

- **HTTP = HyperText Transfer Protocol** — Web의 application-layer 프로토콜
- 1990년대 초중반 탄생

### Client-Server 모델

| 역할 | 누가 | 뭘 하나 |
|------|------|--------|
| Client | 브라우저 (Safari, Firefox, Chrome 등) | HTTP로 object를 요청하고 받아서 표시 |
| Server | 웹 서버 (Apache, IIS 등) | HTTP로 요청에 맞는 object를 응답 |

### HTTP는 TCP를 사용한다

- 웹 문서, 이미지 등은 내용이 누락되면 안 됨 → **reliable transport 필수** → TCP 선택
- 동작 순서: client가 TCP 연결 열기 (port 80) → 서버가 수락 → HTTP 메시지 교환 → TCP 연결 종료

### HTTP는 Stateless다

- **Stateless:** 서버가 이전 요청에 대한 정보를 기억하지 않음, 매 요청이 독립적

**왜 stateless로 설계했나:**
- 수백만 명의 접속 기록을 전부 관리하면 서버에 엄청난 부담
- crash 시 양쪽 state 불일치 → 복구 복잡
- stateless면 매 요청을 독립적으로 처리하면 되므로 서버가 단순하고 가벼움

**주의:** stateless인데 로그인이 유지되는 이유는 **Cookie**라는 보조 메커니즘 덕분 (HTTP 프로토콜 자체는 여전히 stateless)


---

## 3. HTTP Connections

### Non-persistent HTTP (HTTP 1.0)

**규칙: object 하나당 TCP 연결 하나. 받으면 바로 끊는다.**

```
[base HTML 받기] TCP 열기 → 요청 → 응답 → TCP 닫기
[JPEG #1 받기]   TCP 열기 → 요청 → 응답 → TCP 닫기
[JPEG #2 받기]   TCP 열기 → 요청 → 응답 → TCP 닫기
... (referenced object 수만큼 반복)
```

### 응답 시간 계산 ⭐

**RTT (Round Trip Time):** 작은 패킷이 client → server → client로 왕복하는 시간

object 하나를 받는 데 걸리는 시간:

| 단계 | 소요 시간 |
|------|---------|
| TCP 연결 설정 (3-way handshake) | 1 RTT |
| HTTP request 전송 + response 첫 바이트 수신 | 1 RTT |
| 파일 데이터 전송 | file transmission time |

**∴ Non-persistent HTTP response time = 2RTT + file transmission time**

- 3-way handshake의 3번째 ACK에 HTTP request를 함께 실어보낼 수 있어서 2RTT
- 여러 object를 받아야 하면 **object마다 이 과정을 반복** → 비효율적

### Non-persistent HTTP의 문제점

1. **object당 2RTT** — 시간 낭비
2. **OS overhead** — TCP 연결마다 소켓, 버퍼 등 자원 할당/해제 필요
3. 브라우저가 **여러 병렬 TCP 연결**을 동시에 열어 꼼수를 쓰지만 근본적 해결 아님

### Persistent HTTP (HTTP 1.1)

**핵심: TCP connection을 계속 열어둔다.**

| | Non-persistent | Persistent |
|---|---|---|
| TCP 연결 | object마다 열고 닫음 | 열어둔 채로 여러 object 전송 |
| 완료 후 | 즉시 닫음 | 원하는 것 다 받고 나서 닫음 |
| 효율 | object당 2RTT | 모든 referenced object에 대해 최소 1RTT |

**추가 개선: Pipelining**
- referenced object를 발견하는 즉시 응답을 기다리지 않고 연속으로 요청 전송 가능
- 모든 referenced object에 대해 1RTT 정도면 충분 → 응답 시간 절반 가까이 감소

### RTT 비교

| | 첫 번째 object (base HTML) | 나머지 referenced objects |
|---|---|---|
| Non-persistent | 2RTT + transmission | **각각** 2RTT + transmission |
| Persistent | 2RTT + transmission (연결 설정 포함) | **각각** 1RTT + transmission (연결 이미 있음) |

### 핵심 구분

- **병렬 TCP 연결 ≠ Persistent:** 병렬 연결은 Non-persistent의 꼼수 (여전히 열고 닫음), Persistent는 근본적으로 하나의 연결을 유지
- **Persistent connection ≠ Stateful:** TCP 연결은 유지하지만 HTTP는 여전히 stateless — 서버는 이전 요청을 기억하지 않음

---

## 앞에서 배운 개념과의 연결

| 이전 개념 | HTTP에서 어떻게 나타나는지 |
|---------|---------------------|
| Client-Server | 브라우저(client) ↔ 웹 서버(server) |
| Socket | 브라우저가 소켓을 만들어 서버 port 80에 연결 |
| IP + Port | 서버 IP + port 80 (HTTP 기본 포트) |
| TCP 선택 이유 | 웹 문서는 no loss 필수 → reliable transport 필요 |
| Stateless | HTTP는 이전 요청을 기억하지 않음 |
| Protocol defines | HTTP가 정의하는 message types, syntax, semantics, rules |

---

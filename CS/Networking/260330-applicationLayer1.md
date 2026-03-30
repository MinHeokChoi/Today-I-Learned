# TIL: Application Layer — Principles of Network Applications

> **범위:** Chapter 2 — Application Layer 전반부 (슬라이드 2-1 ~ 2-16)  

---

## 1. Creating a Network App

네트워크 앱 개발자는 **end system(끝단)만 신경 쓰면 된다.**

- 라우터 같은 네트워크 코어 장비에는 application 계층이 존재하지 않음
- 코어 장비는 패킷 전달(forwarding)만 수행하며, user application을 실행하지 않음
- 개발자는 소켓 API를 통해 transport 계층에 메시지를 맡기면 그 아래는 OS와 네트워크 인프라가 처리
- 이 덕분에 end system용 소프트웨어만 작성하면 즉시 배포 가능 → **신속한 앱 개발, 인터넷 생태계의 폭발적 성장**

---

## 2. Client-Server Paradigm

역할이 **고정**된 구조.

### Server
- **Always-on:** 클라이언트가 언제 접속할지 모르므로 365일 가동
- **Permanent IP address:** IP가 바뀌면 클라이언트가 서버를 찾을 수 없음
- 트래픽 증가 시 data center로 scaling 가능

### Client
- 간헐적 접속 (필요할 때만)
- 동적 IP 가능
- **클라이언트끼리 직접 통신하지 않음** — 반드시 서버를 거침

### 대표 예시
HTTP, IMAP, FTP 등 우리가 다루는 거의 모든 프로토콜

---

## 3. Peer-to-Peer (P2P) Paradigm

역할이 **고정되지 않는** 구조. 모든 참여자가 client도 되고 server도 됨.

### 핵심 특징
- **No always-on server:** 중앙 서버 불필요
- **Self scalability:** 새 peer가 demand와 capacity를 동시에 가져옴 → 사용자 증가 시 시스템 전체 능력도 함께 성장
- **간헐적 접속 + 동적 IP** → complex management
- **Incentive mechanism 필수:** free-rider(받기만 하고 나가는 peer) 문제 방지

### 대표 예시
BitTorrent (tit-for-tat 방식으로 incentive 강제), 블록체인 (탈중앙화 목적)

### Client-Server vs P2P 선택 기준

| 기준 | Client-Server 유리 | P2P 유리 |
|------|-------------------|---------|
| 서비스 안정성 | 일관된 품질 보장 필요 | 다소 불안정해도 OK |
| 확장성 비용 | 사용자 적거나 예측 가능 | 사용자 폭발적 증가 예상 |
| 중앙 통제 | 인증, 과금 등 제어 필요 | 탈중앙화가 목적 |

---

## 4. Processes Communicating

### 기본 개념
- **Process:** 호스트 안에서 실행 중인 프로그램
- 같은 호스트 내: **IPC(Inter-Process Communication)** — OS 영역
- 다른 호스트 간: **메시지(message) 교환** — 네트워킹의 핵심

### Client Process vs Server Process
- **Client process:** 통신을 시작(initiate)하는 쪽
- **Server process:** 요청을 기다리는(wait) 쪽
- 이 구분은 "첫 연결 기준"이며, 프로토콜에 관계없이 동일
- **P2P에서도 client process와 server process가 둘 다 존재** — 역할이 고정되지 않을 뿐

---

## 5. Sockets

프로세스와 네트워크 사이의 **문(door)**.

- 프로세스가 소켓에 메시지를 밀어넣으면, 문 바깥은 OS/네트워크가 처리
- 통신 한 건에 **소켓 두 개** 필요 (송신 측 + 수신 측)
- 소켓은 프로토콜이 아니라 **인터페이스(API)** — 인터넷에서 가장 유명한 API

### 앱 개발자의 제어 범위

| 영역 | 제어 주체 |
|------|---------|
| 문 안쪽 (메시지 내용, TCP/UDP 선택, 목적지 지정) | **앱 개발자** |
| 문 바깥 (라우팅 경로, 패킷 전달 등) | **OS / 네트워크** |

---

## 6. Addressing Processes

**프로세스 식별 = IP 주소 + 포트 번호**

- IP 주소만으로는 "이 컴퓨터의 어떤 프로세스인지" 특정 불가
- IP 주소: 호스트(정확히는 네트워크 인터페이스) 식별 — 32비트
- 포트 번호: 해당 호스트 내 특정 프로세스 식별 — 16비트 (0~65535)

### Well-known 포트 번호

| 서비스 | 포트 번호 |
|--------|----------|
| HTTP | 80 |
| HTTPS | 443 |
| SMTP | 25 |
| DNS | 53 |

### 주의사항
- IP 주소는 호스트가 아닌 **NIC(네트워크 인터페이스)**에 부여됨 → 한 컴퓨터에 복수 IP 가능
- 클라이언트에도 포트 번호 존재 — OS가 ephemeral port(49152~65535)를 자동 배정
- IP 주소 표기: dotted decimal (예: 128.119.245.12) — 32비트를 8비트씩 끊어 10진수로 표기

---

## 7. Application-Layer Protocol Defines

프로토콜은 **네 가지**를 정의해야 한다:

| 항목 | 의미 | 예시 (HTTP) |
|------|------|------------|
| **Message types** | 메시지 종류 | request, response |
| **Message syntax** | 메시지 형식 (field 배치, 길이) | `GET /index.html HTTP/1.1` — 메서드, URL, 버전 순서 |
| **Message semantics** | field 값의 의미 | 상태코드 200 = 성공, 404 = 못 찾음 |
| **Rules** ⭐ 가장 핵심 | 언제 어떻게 보내고 응답하는지 | 클라이언트가 먼저 request, 서버가 response |

### Open vs Proprietary Protocol

| 구분 | Open Protocol | Proprietary Protocol |
|------|--------------|---------------------|
| 정의 문서 | RFC (IETF 작성) | 비공개 |
| 장점 | 상호운용성 (interoperability) | 자사 IP 보호 |
| 예시 | HTTP, SMTP | Skype, Zoom |

### 핵심 구분
- **Application ≠ Application-layer protocol** (Web은 서비스, HTTP는 프로토콜)
- **Syntax ≠ Semantics** (syntax: "어떻게 생겼나", semantics: "무슨 뜻이나")

---

## 8. What Transport Service Does an App Need?

앱이 transport 계층에 요구하는 서비스는 네 가지 기준으로 분류:

| 기준 | 설명 | 예시 |
|------|------|------|
| **Data integrity** | 데이터 손실 없이 전달 보장 | 파일 전송, 웹 — no loss 필수 / 오디오 — loss-tolerant |
| **Timing** | 낮은 지연시간 | 인터넷 전화, 게임 — 10's msec 이내 필수 |
| **Throughput** | 최소 대역폭 보장 | 영상 스트리밍 — minimum 필요 / 이메일 — elastic |
| **Security** | 암호화, 데이터 무결성 | 인터넷 뱅킹 — 필수 (TLS) |

**주의:** 하나의 앱 안에서도 데이터 종류에 따라 요구사항이 다를 수 있음 (예: 실시간 코칭 앱 — 영상은 loss-tolerant + timing 중요, 운동 기록은 no loss + timing 덜 중요)

---

## 9. Internet Transport Protocols: TCP vs UDP

| | TCP | UDP |
|---|-----|-----|
| **신뢰성** | ✅ reliable transport | ❌ unreliable (최선을 다하지만 보장 못 함) |
| **흐름 제어** | ✅ sender가 receiver를 overwhelm하지 않도록 | ❌ |
| **혼잡 제어** | ✅ 네트워크 과부하 시 sender throttle | ❌ |
| **연결** | ✅ connection-oriented (setup 필요) | ❌ 연결 개념 없음 |
| **미제공** | timing, throughput 보장, security | 위 전부 + 신뢰성까지 미제공 |

### UDP가 존재하는 이유
- TCP의 오버헤드(연결 설정, 흐름/혼잡 제어)를 피할 수 있음
- timing에 민감하고 약간의 loss를 허용하는 앱에 적합

### Securing TCP: TLS
- TCP 자체에는 암호화 없음 — 평문이 그대로 인터넷을 통과
- **TLS(Transport Layer Security):** application 계층에서 구현되며, TCP 위에 암호화 추가
- TLS를 적용한 HTTP = **HTTPS**
- 제공: 암호화 + 데이터 무결성 + end-point 인증

---

## 퀴즈에서 틀리거나 보완한 것들

1. **Client-Server의 client도** 간헐적 접속 + 동적 IP 특성을 가짐 (P2P만의 특징이 아님)
2. 앱 개발자가 제어 가능한 것: 메시지 내용, TCP/UDP 선택, **목적지 IP+port 지정까지 포함**
3. **Stateless의 정확한 의미:** 서버가 이전 요청의 상태를 기억하지 않음 — 매 요청이 독립적. Cookie로 보완 가능하지만 HTTP 프로토콜 자체는 stateless
4. **Client-Server 패러다임 ≠ Stateful:** 구조(패러다임)와 프로토콜 특성은 별개 개념
5. **TCP 연결 기반 통신:** 연결이 맺어진 후에는 매번 IP/port를 지정할 필요 없음 (UDP와의 차이)

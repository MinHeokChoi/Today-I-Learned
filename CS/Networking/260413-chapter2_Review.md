# 네트워킹 기초 — HTTP, HTTPS, DNS, Email, SMTP, IMAP

> Kurose & Ross 8th ed. Chapter 2 학습 정리  
> 학습 날짜: 2026-04-14

---

## 목차

1. [HTTP / HTTPS](#1-http--https)
2. [DNS](#2-dns)
3. [Email, SMTP, IMAP](#3-email-smtp-imap)

---

## 1. HTTP / HTTPS

### HTTP란?

HTTP(HyperText Transfer Protocol)는 웹에서 데이터를 주고받기 위한 프로토콜.  
항상 **클라이언트가 먼저 요청(Request)** 을 보내고, 서버가 응답(Response)을 돌려주는 구조.

### HTTP 요청 구조

```
GET /index.html HTTP/1.1       ← 메서드 + URL + 버전
Host: www.example.com          ← 헤더
User-Agent: Chrome/120

(GET은 바디 없음)
```

### 주요 HTTP 메서드

| 메서드 | 용도 | 멱등성 | 캐싱 | 바디 |
|---|---|---|---|---|
| GET | 조회 | ✓ | ✓ | ✗ |
| POST | 생성 | ✗ | ✗ | ✓ |
| PUT | 전체 교체 | ✓ | ✗ | ✓ |
| DELETE | 삭제 | ✓ | ✗ | 비권장 |

**GET** — 파라미터가 URL에 노출됨 (`/search?q=keyword`). 북마크·공유 가능. 로그인 정보에 부적합.  
**POST** — 데이터를 바디에 담아 전송. URL에 노출 안 됨. 단, 바디에 담는다고 자동 암호화는 아님 → HTTPS 필수.  
멱등성(idempotent): 같은 요청을 여러 번 보내도 결과가 동일해야 함. POST는 같은 요청을 두 번 보내면 데이터가 두 번 생성될 수 있음.  
PUT은 전체 교체, 일부만 수정할 때는 **PATCH** 사용.  
DELETE 바디: 표준(RFC 7231)에서 허용은 하지만 비권장. 많은 서버·클라이언트가 무시하므로 실제로는 사용하지 않음.

### HTTP 응답 상태 코드

| 범위 | 의미 | 예시 |
|---|---|---|
| 2xx | 성공 | 200 OK, 201 Created |
| 3xx | 리다이렉트 | 301 Moved, 304 Not Modified |
| 4xx | 클라이언트 오류 | 404 Not Found |
| 5xx | 서버 오류 | 500 Internal Server Error |

### HTTPS = HTTP + TLS 암호화

HTTPS는 HTTP에 TLS(Transport Layer Security) 암호화를 추가한 것.  
중간에서 패킷을 탈취해도 복호화 불가.

### TLS 핸드셰이크 과정 (TLS 1.2 기준)

```
클라이언트                              서버
    │                                    │
    │── ① ClientHello ─────────────────>│  TLS 버전, 암호 스위트, Client Random
    │                                    │
    │<─ ② ServerHello ──────────────────│  선택된 암호 스위트, Server Random
    │<─ ③ Certificate ──────────────────│  CA 서명된 인증서 + 공개키
    │<─ ④ ServerHelloDone ──────────────│
    │                                    │
    │   (인증서 검증 + 세션 키 생성)      │
    │                                    │
    │── ⑤ ClientKeyExchange ──────────>│  pre-master secret (공개키로 암호화)
    │── ⑥ 🔒 Finished ────────────────>│  세션 키로 암호화
    │<─ ⑦ 🔒 Finished ──────────────────│
    │                                    │
    │═══════ 이후 HTTP 통신 암호화 ═══════│
```

**핵심 포인트**
- 비대칭 암호화(공개키/개인키)로 pre-master secret을 안전하게 전달
- 이후 실제 데이터는 빠른 **대칭 세션 키**로 암호화 → 비대칭은 느리기 때문
- TLS 1.2는 핸드셰이크에 **2 RTT** 소요
- 인증서 검증: 브라우저 내장 CA 공개키로 서명 유효성 확인

### TLS 1.3의 변화

TLS 1.3은 핸드셰이크 구조 자체가 TLS 1.2와 다름.

```
클라이언트                              서버
    │                                    │
    │── ① ClientHello ─────────────────>│  TLS 버전 + 키 공유 파라미터(ECDHE) 포함
    │                                    │
    │<─ ② ServerHello ──────────────────│  키 공유 파라미터 응답 → 세션 키 합의 완료
    │<─ ③ 🔒 Certificate + Finished ───│  이미 암호화된 상태로 전송
    │                                    │
    │── ④ 🔒 Finished ────────────────>│
    │                                    │
    │═══════ 이후 HTTP 통신 암호화 ═══════│
```

| | TLS 1.2 | TLS 1.3 |
|---|---|---|
| 핸드셰이크 | 2 RTT | **1 RTT** |
| 재연결 | 1 RTT | **0-RTT** 가능 |
| 키 교환 | RSA 또는 ECDHE | **ECDHE만 허용** |
| 전방향 비밀성 | 선택적 | **항상 보장** |

- **전방향 비밀성(Forward Secrecy)**: 나중에 서버 개인키가 탈취되더라도 과거 세션 내용은 복호화 불가. ECDHE는 매 세션마다 임시 키를 생성하기 때문.
- RSA 키 교환을 완전 제거 → 과거 트래픽 녹화 후 나중에 해독하는 공격 원천 차단

### HTTP 버전 비교

| 버전 | 특징 |
|---|---|
| HTTP/1.1 | 요청 하나씩 순서대로 처리 |
| HTTP/2 | 하나의 TCP 연결로 멀티플렉싱. 단, TCP HOL Blocking 존재 |
| HTTP/3 | UDP 기반 QUIC 사용. HOL Blocking 해결, 0-RTT 재연결 |

### HTTP/3 & QUIC

**QUIC이 나온 이유 — TCP의 3가지 한계**

1. **HOL Blocking**: HTTP/2가 멀티플렉싱을 해도 TCP 레이어에서 패킷 하나 손실 시 모든 스트림 블로킹
2. **연결 지연**: TCP(1 RTT) + TLS(1 RTT) = 최소 2 RTT 후에야 첫 데이터 전송 가능
3. **경직성**: TCP는 OS 커널에 박혀 있어 수정하려면 전 세계 OS 패치 필요

**QUIC의 해결책**

- UDP 위에서 동작 → OS 커널을 거치지 않아 앱 레이어에서 자유롭게 개선 가능
- 스트림별 독립 처리 → 패킷 손실이 해당 스트림에만 영향, 나머지는 정상 진행
- TLS 1.3 내장 → 연결과 암호화를 동시에 처리, **1 RTT**로 단축
- **Connection ID** 도입 → IP가 바뀌어도 (WiFi → LTE) 연결 유지 (Connection Migration)
- 재연결 시 **0-RTT** 가능

```
HTTP/2 프로토콜 스택          HTTP/3 프로토콜 스택
┌─────────────┐               ┌─────────────┐
│   HTTP/2    │               │   HTTP/3    │
├─────────────┤               ├─────────────┤
│     TLS     │               │    QUIC     │  ← TLS 1.3 내장
├─────────────┤               ├─────────────┤
│     TCP     │               │     UDP     │
├─────────────┤               ├─────────────┤
│     IP      │               │     IP      │
└─────────────┘               └─────────────┘
```

> iOS 연관: `URLSession`은 iOS 14부터 HTTP/3 / QUIC 자동 지원

---

## 2. DNS

### DNS란?

DNS(Domain Name System)는 도메인 이름(`www.google.com`)을 IP 주소(`142.250.196.68`)로 변환하는 인터넷의 전화번호부.

### DNS 계층 구조

```
루트 (.)
├── .com
│   ├── google.com  → ns1.google.com (권한 네임서버)
│   │   ├── www.google.com
│   │   └── mail.google.com
│   └── naver.com
└── .kr
    └── ...
```

도메인은 **오른쪽이 상위, 왼쪽이 하위**.

```
www   .   example   .   com   .
 ↑           ↑          ↑     ↑
서브도메인   SLD(2차)   TLD   루트(.)
```

- **Apex 도메인 (루트 도메인)**: `example.com` — 내가 구입한 도메인 자체
- **서브도메인**: `www.example.com`, `mail.example.com` — Apex 앞에 붙는 것

### Iterated Query (반복 질의) — 실제 인터넷 표준

```
브라우저
  ↓ ① 질의
Resolver (8.8.8.8 등)  ← Local DNS Name Server
  ↓ ② 루트에 질의
루트 서버
  ↓ ③ ".com TLD 서버 주소" 반환 (referral)
Resolver
  ↓ ④ TLD 서버에 질의
.com TLD 서버
  ↓ ⑤ "google.com 권한 서버 주소" 반환 (referral)
Resolver
  ↓ ⑥ 권한 서버에 질의
권한 네임서버 (ns1.google.com)
  ↓ ⑦ A 레코드 IP 반환
Resolver
  ↓ ⑧ IP 캐시 저장 후 브라우저에 전달
```

**Iterated vs Recursive**
- **Iterated** (실제 표준): Resolver가 직접 각 서버를 순서대로 방문. 각 서버는 referral만 반환
- **Recursive** (사실상 미사용): 각 서버가 다음 서버에 위임해 최종 답 반환. 루트 서버 과부하 유발 → 대부분의 루트/TLD 서버가 거부
- 예외: 클라이언트(브라우저) → Resolver 구간은 recursive 허용

> `Resolver` = `Local DNS Name Server` (Kurose & Ross 교재 표현) = `Recursive Resolver` (RFC 표준)

### DNS 캐싱과 TTL

각 레코드에는 TTL(Time To Live) 값이 있어 그 시간 동안 캐시에 저장해 재사용.

```
캐시 레이어 (순서대로 확인):
브라우저 캐시 → OS 캐시(/etc/hosts) → Resolver 캐시 → 권한 서버
```

- TTL이 짧으면: 변경 빠르게 반영 but DNS 조회 트래픽 증가
- TTL이 길면: 캐시 효율 높음 but 변경 반영 느림
- 서버 이전 시 미리 TTL을 낮춰두는 것이 관례

### DNS 레코드 타입

**레코드 구문**
```
이름                TTL    클래스  타입   값
www.example.com.   300    IN      A      93.184.216.34
```

| 타입 | 역할 | 예시 |
|---|---|---|
| A | 도메인 → IPv4 | `www.example.com. → 93.184.216.34` |
| AAAA | 도메인 → IPv6 | `www.example.com. → 2606:2800::` |
| NS | 담당 네임서버 위임 | `google.com. → ns1.google.com.` |
| CNAME | 도메인 별명 | `www.example.com. → example.com.` |
| MX | 메일 수신 서버 | `example.com. → mail.example.com. (priority 10)` |
| TXT | 자유형식 텍스트 | SPF, DKIM, 소유권 인증 등 |

**A 레코드**
- 도메인을 IPv4 주소로 변환. DNS 조회의 최종 목적지
- 하나의 도메인에 여러 A 레코드 등록 가능 → 자동 로드밸런싱

**NS 레코드**
- "이 도메인 질문은 저 서버가 담당해" 라는 하향 위임 선언
- Iterated query의 각 단계를 연결하는 표지판
- NS가 가리키는 서버의 IP(글루 레코드)도 함께 제공됨
- TTL이 길다 (172800초 = 48시간) — 자주 바뀌지 않으므로

**CNAME 레코드**
- 한 도메인을 다른 도메인의 별명으로 만듦
- 항상 최종적으로 A 레코드(IP)에 도달할 때까지 체인을 따라감
- **Apex 도메인에 사용 불가** — RFC 1034에 따라 CNAME이 있는 이름에는 다른 레코드가 공존할 수 없음. Apex 도메인에는 SOA·NS 레코드가 반드시 있어야 하므로 구조적으로 불가능
- **서브도메인에만 사용** (`www.example.com`, `blog.example.com` 등)
- CDN, GitHub Pages, Vercel 등 외부 서비스 연결에 자주 활용

```
; Apex 도메인 → A 레코드 (IP 직접)
example.com.      IN  A      93.184.216.34     ✓

; 서브도메인 → CNAME (다른 도메인으로 연결)
www.example.com.  IN  CNAME  example.com.      ✓
blog.example.com. IN  CNAME  example.github.io. ✓
```

**MX 레코드**
- 해당 도메인으로 오는 이메일을 받을 서버 지정
- 우선순위(priority) 숫자 포함 — 숫자가 작을수록 먼저 시도
- 여러 개 등록으로 백업 서버 구성 가능

```
example.com.  IN  MX  10  mail1.example.com.  ← 1순위
example.com.  IN  MX  20  mail2.example.com.  ← 2순위 (백업)
```

**TXT 레코드**
- 자유형식 텍스트. 현재는 주로 기계가 읽는 용도로 활용
- SPF, DKIM, DMARC (이메일 위조 방지)
- 도메인 소유권 인증 (Google Search Console 등)
- 여러 개 동시 존재 가능

### DNS 포트

| 상황 | 프로토콜 | 포트 |
|---|---|---|
| 일반 조회 (512바이트 이하) | UDP | 53 |
| 응답이 512바이트 초과 시 | TCP | 53 |
| 존 전송 (Zone Transfer) | TCP | 53 |

> DNS는 기본적으로 UDP를 사용하지만, 응답 크기가 크거나 존 전송(서버간 레코드 전체 복사) 시에는 TCP로 전환.

### DNSSEC

기본 DNS는 응답이 위조될 수 있음 (DNS 스푸핑 / 캐시 포이즈닝).  
DNSSEC은 각 레코드에 디지털 서명을 추가해 응답 무결성 검증.

- **DoH (DNS over HTTPS)** / **DoT (DNS over TLS)**: DNS 질의 자체를 암호화

### macOS DNS 확인 명령어

```bash
# 현재 DNS 설정 확인
scutil --dns

# DNS 조회
nslookup naver.com

# 출력 해석:
# Server: 203.249.66.1      ← Resolver (Local DNS Name Server)
# Non-authoritative answer  ← Resolver 캐시에서 응답 (권한 서버 아님)
# Address: 223.130.200.236  ← A 레코드 (여러 개면 로드밸런싱)

# 권한 서버에 직접 질의
nslookup naver.com ns1.naver.com
```

---

## 3. Email, SMTP, IMAP

### 이메일 시스템 구조

이메일은 단일 프로토콜이 아니라 역할별로 분리된 시스템.

```
송신자 ──SMTP──> 송신 서버 ──SMTP──> 수신 서버 <──IMAP── 수신자
                    │                    ↑
                    └── DNS MX 조회 ─────┘
```

### SMTP (Simple Mail Transfer Protocol)

- 메일 **전송** 담당 (클라이언트→서버, 서버→서버 모두)
- 포트: 587 (클라이언트→서버, TLS), 25 (서버간), 465 (SSL)

```
C: EHLO gmail.com
S: 250 OK
C: MAIL FROM:<alice@gmail.com>
S: 250 OK
C: RCPT TO:<bob@example.com>
S: 250 OK
C: DATA
S: 354 Start input
C: Subject: 안녕하세요
C:
C: 본문 내용입니다.
C: .                    ← 한 줄에 점(.) 하나 = 본문 종료 신호 (RFC 5321)
S: 250 Message accepted
C: QUIT
```

### SMTP 포트 25 주의사항

포트 25는 서버 간 전송 표준 포트이나, 스팸 방지 목적으로 **대부분의 ISP와 클라우드(AWS, GCP 등)는 아웃바운드 포트 25를 기본 차단**함. 실제 운영 환경에서는 587(STARTTLS) 또는 465(SSL) 사용이 일반적.

### DNS MX 레코드로 수신 서버 찾기

SMTP 서버가 `bob@example.com` 으로 메일을 보낼 때:
1. `example.com` 의 MX 레코드를 DNS에서 조회
2. 반환된 메일 서버 주소로 SMTP 연결

### IMAP (Internet Message Access Protocol)

- 메일 **읽기** 담당. 서버 메일박스에 직접 접근
- 포트: 993 (TLS)
- 메일을 서버에 보존 → 폰·PC·웹 어디서든 동기화
- 읽음·삭제·폴더 상태가 모든 기기에 반영

**IMAP vs POP3**

| | IMAP | POP3 |
|---|---|---|
| 메일 보관 | 서버에 유지 | 다운로드 후 삭제 (기본) |
| 다기기 동기화 | ✓ | ✗ |
| 포트 (TLS) | 993 | 995 |
| 현재 사용 | 주류 | 거의 미사용 |

### 이메일 보안 — SPF / DKIM / DMARC

기본 이메일은 발신자를 속일 수 있어 (스푸핑) 3가지 DNS 기반 인증 메커니즘 사용.

| | 역할 | 검증 방법 |
|---|---|---|
| SPF | 허가된 IP에서 보낸 건가? | DNS TXT 레코드의 IP 목록 대조 |
| DKIM | 내용이 변조되지 않았나? | 개인키 서명 → DNS 공개키로 검증 (비대칭 암호화) |
| DMARC | 실패 시 어떻게 처리? | none / quarantine / reject 정책 |

```
; SPF
example.com.  IN  TXT  "v=spf1 include:_spf.google.com ~all"

; DMARC
_dmarc.example.com.  IN  TXT  "v=DMARC1; p=reject; rua=mailto:dmarc@example.com"
```

### 전체 흐름 (alice → bob)

```
① alice가 메일 작성 → SMTP(587)로 Gmail 서버에 전달
② Gmail 서버가 DNS MX 조회 → example.com 수신 서버 주소 획득
③ Gmail 서버 → 수신 서버로 SMTP 전송 (SPF·DKIM 서명 포함)
④ 수신 서버가 DMARC 검증 후 bob의 메일박스에 저장
⑤ bob이 IMAP으로 접속 → 어느 기기에서든 동일한 메일함 확인
```

---

## 핵심 포트 정리

| 프로토콜 | 포트 | 용도 |
|---|---|---|
| HTTP | 80 | 웹 (평문) |
| HTTPS | 443 | 웹 (TLS 암호화) |
| DNS | 53 (UDP) | DNS 일반 조회 |
| DNS | 53 (TCP) | DNS 대용량 응답 / 존 전송 |
| SMTP | 587 | 메일 전송 (클라이언트→서버, STARTTLS) |
| SMTP | 25 | 메일 전송 (서버간, 클라우드에서 기본 차단) |
| SMTP | 465 | 메일 전송 (SSL) |
| IMAP | 993 | 메일 수신 (TLS) |
| POP3 | 995 | 메일 수신 (TLS, 구식) |

---

## 개념 간 연결 관계

```
브라우저에 URL 입력
    │
    ├─ DNS 조회 (Iterated Query)
    │      └─ NS 레코드로 위임 → A 레코드로 IP 획득
    │
    ├─ TCP 연결 (3-way 핸드셰이크)
    │
    ├─ TLS 핸드셰이크 (HTTPS인 경우)
    │      ├─ TLS 1.2: 비대칭으로 키 교환 → 대칭 세션 키로 암호화 (2 RTT)
    │      └─ TLS 1.3: ECDHE로 1 RTT 단축, 전방향 비밀성 항상 보장
    │
    └─ HTTP 요청/응답

이메일 전송
    │
    ├─ DNS MX 조회 → 수신 서버 탐색
    ├─ SMTP 전송 (SPF·DKIM 서명 포함)
    └─ IMAP으로 수신함 확인
```

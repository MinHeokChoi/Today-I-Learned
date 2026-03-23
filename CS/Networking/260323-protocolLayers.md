# 컴퓨터 네트워크 Ch.1 — Protocol Layers & Encapsulation

> Kurose & Ross, *Computer Networking: A Top-Down Approach* 8th Edition  
> 슬라이드 69~79페이지 / 교재 1.5절 내용 정리

---

## 1. 왜 계층화(Layering)를 하는가?

인터넷은 호스트, 라우터, 링크, 프로토콜, 앱 등 수많은 구성요소가 얽혀 있는 복잡한 시스템이다.  
이걸 계층으로 나누면 두 가지 이점이 있다.

- **구조화**: 시스템의 각 부분과 관계를 명확히 파악할 수 있다
- **모듈화**: 한 계층을 바꿔도 나머지 계층에 영향이 없다 (예: Ethernet → WiFi로 바꿔도 application은 무관)

---

## 2. 5계층 인터넷 프로토콜 스택

| 계층 | 역할 | 전송 단위 | 대표 프로토콜 |
|------|------|----------|-------------|
| **Application** | 네트워크 앱 지원 | message | HTTP, SMTP, DNS |
| **Transport** | 프로세스 ↔ 프로세스 전달 | segment | TCP, UDP |
| **Network** | 호스트 → 호스트 경로 결정 | datagram | IP, 라우팅 프로토콜 |
| **Link** | 인접 노드 간 전달 | frame | Ethernet, WiFi, PPP |
| **Physical** | 실제 비트를 물리적 매체로 전송 | bits | — |

### 핵심 원리: 각 계층은 아래 계층의 서비스를 이용해서 위 계층에 서비스를 제공한다

네이버 접속을 예로 들면:

1. **Application**: "네이버 페이지 가져와" HTTP message 생성 → transport에게 "전달해줘" 맡김
2. **Transport**: Ht 헤더 붙여 segment 생성 (포트 번호 포함) → network에게 "이 IP로 보내줘" 맡김
3. **Network**: Hn 헤더 붙여 datagram 생성 (출발지/목적지 IP) → link에게 "다음 노드로 보내줘" 맡김
4. **Link**: Hl 헤더 붙여 frame 생성 (MAC 주소) → physical에게 "비트로 바꿔서 보내줘" 맡김
5. **Physical**: 비트를 전기 신호/빛으로 변환해서 실제 매체로 전송

각 계층은 **자기 일만 하고 나머지는 아래에 맡긴다.** 위/아래 계층이 내부적으로 어떻게 처리하는지는 신경 쓰지 않는다.

---

## 3. Transport 계층 상세

프로세스와 프로세스 사이의 데이터 배달부 역할.  
Network 계층이 컴퓨터 A → 컴퓨터 B까지 보내주는 거라면, Transport는 그 컴퓨터 안의 **어떤 앱**에 전달할지까지 책임진다.

### TCP vs UDP

| | TCP | UDP |
|---|---|---|
| 연결 | 연결 지향 (connection-oriented) | 비연결 (connectionless) |
| 신뢰성 | 보장 (guaranteed delivery) | 보장 없음 |
| 흐름 제어 | 있음 (sender/receiver 속도 맞춤) | 없음 |
| 혼잡 제어 | 있음 (네트워크 혼잡 시 속도 줄임) | 없음 |
| 용도 | 웹, 이메일, 파일 전송 | 실시간 스트리밍, 게임 |

---

## 4. Network 계층과 Link 계층의 관계

### 택배 비유로 이해하기

- **Network 계층(IP)** = 택배 송장: 출발지/목적지 주소가 적혀 있고, **출발부터 도착까지 변하지 않음**
- **Link 계층** = 구간별 운송 수단: 구간마다 바뀔 수 있음 (트럭 → 비행기 → 또 다른 트럭)

### 라우터에서 실제로 일어나는 일

홍대 내부(Ethernet) → 홍대 라우터 → KT(PPP)로 패킷이 나갈 때:

```
① Ethernet frame 도착: [Hl_ethernet | Hn | Ht | M]
② 라우터가 Ethernet 헤더(Hl) 벗김
③ IP 헤더(Hn) 확인 → "이건 KT 방향으로 보내야겠다" 판단
④ PPP 헤더(Hl')를 새로 씌움: [Hl_ppp | Hn | Ht | M]
⑤ PPP 링크로 전송
```

**link 헤더는 매 구간마다 벗기고 다시 붙이지만, IP datagram은 그대로 유지된다.**  
이것이 IP가 "the glue that binds the Internet together"(접착제)라고 불리는 이유.

---

## 5. 네트워크 구성요소: 스위치 vs 라우터

### 노드(node)란?

네트워크에서 데이터가 지나가는 모든 지점을 통칭. 호스트, 스위치, 라우터 모두 노드다.

### 스위치와 라우터의 역할 차이

| | 스위치 | 라우터 |
|---|---|---|
| **위치** | 한 네트워크 **안에서** 노드들을 연결 | 네트워크와 네트워크 **사이의 경계** |
| **계층** | Link + Physical (2계층) | Network + Link + Physical (3계층) |
| **보는 것** | frame 헤더 (MAC 주소) | IP 헤더 (IP 주소) |
| **하는 일** | 같은 네트워크 내에서 빠르게 전달 | IP 주소 보고 다음 네트워크 방향 결정 |

### 실제 구조 예시

```
[홍대 내부 네트워크 - Ethernet]              [KT 네트워크]

PC ── 스위치 ── 스위치 ── 라우터 ═══ 라우터 ═══ ...
       |           |         ↑
     프린터      노트북     경계
                        (두 네트워크에
                         동시에 발을
                         걸치고 있음)
```

- **라우터의 특별한 점**: 두 개 이상의 서로 다른 네트워크에 동시에 발을 걸치고 있어서 둘 사이를 이어줄 수 있다

---

## 6. Encapsulation (캡슐화)

데이터가 한 계층씩 내려갈 때마다 각 계층의 헤더가 붙는다.  
마트료시카 인형처럼 겹겹이 감싸는 구조.

```
Application:                    [ M ]              ← message
Transport:                [ Ht | M ]              ← segment
Network:           [ Hn | Ht | M ]              ← datagram
Link:        [ Hl | Hn | Ht | M ]              ← frame
Physical:    → 비트로 변환하여 전송
```

수신 측에서는 반대로 헤더를 하나씩 벗기면서 올라간다.  
각 계층은 자신의 헤더만 보고 처리한 후, 위 계층으로 올려보낸다.

### 장비별 계층 범위

| 장비 | 보유 계층 | 이유 |
|------|----------|------|
| **호스트** | 5계층 전부 | 데이터를 생성하고 최종 수신 |
| **라우터** | Network + Link + Physical | IP 주소를 봐야 다음 목적지를 결정할 수 있으니까 |
| **스위치** | Link + Physical | frame만 보고 같은 네트워크 내에서 전달하면 되니까 |

---

## 7. 참고: OSI 7계층 모델

인터넷 5계층 스택에는 없지만 OSI 모델에는 2개 계층이 더 있다.

- **Presentation**: 데이터 해석 (암호화, 압축, 기계별 변환)
- **Session**: 동기화, 체크포인트, 데이터 교환 복구

인터넷에서는 이 기능이 필요하면 application 계층에서 직접 구현한다.

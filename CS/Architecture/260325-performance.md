# TIL: Computer Architecture — Performance (Lec 8)

> Chapter 1. Computer Abstractions and Technology  
> Hongik Univ. Computer Architecture

---

## 1. 성능 지표

컴퓨터의 종류와 목적에 따라 중요한 성능 지표가 다르다.

| 지표 | 정의 | 주요 사용처 |
|---|---|---|
| **Response Time** (응답시간) | 작업 시작 ~ 완료까지 걸리는 시간 | PC, 임베디드 |
| **Throughput** (처리량) | 단위 시간당 처리한 작업 수 | 서버, 데이터센터 |

- PC는 **한 사용자가 한 작업을 기다리는** 구조 → Response Time이 체감 성능
- 데이터센터는 **수천 개의 요청을 동시에 소화하는** 구조 → Throughput이 비용 효율과 직결
- 단, 데이터센터도 SLA(응답시간 기준)는 존재하므로 둘 다 고려하되 **우선순위**가 다른 것

---

## 2. 파이프라이닝과 성능 (세탁 예시)

| | Sequential | Pipelined |
|---|---|---|
| Response Time | 90분 | 90분 (동일) |
| Throughput | 0.67 tasks/hr | 1.14 tasks/hr |

**교훈**
- 파이프라이닝은 단일 작업의 응답시간을 줄이지 못한다
- 여러 작업을 겹쳐 처리해 **처리량**을 높인다
- 파이프라인 단계 길이가 불균형하면 속도 향상이 제한된다

---

## 3. 실행시간의 종류

```
Elapsed Time (경과시간)
└── CPU Time + I/O 대기 + OS overhead + ...
    └── 시스템 전체 성능을 나타냄

CPU Time
└── CPU가 해당 프로그램에만 쓴 시간
    └── 성능 비교에 주로 사용
```

---

## 4. 성능과 실행시간의 관계

$$\text{Performance}_X = \frac{1}{\text{Execution Time}_X}$$

$$\frac{\text{Performance}_X}{\text{Performance}_Y} = \frac{\text{Execution Time}_Y}{\text{Execution Time}_X} = n$$

> X가 Y보다 n배 빠르다 = Y의 실행시간 / X의 실행시간

---

## 5. CPU Clock 기초

모든 CPU는 클럭에 맞춰 동작한다.

```
클럭 주기 T  : 한 사이클의 지속 시간   ex) 500ps = 0.5ns
클럭 주파수 f : 초당 사이클 수 (= 1/T)  ex) 1/0.5ns = 2.0GHz
```

---

## 6. CPU Time 공식 (핵심)

### 기본형

$$\text{CPU Time} = \text{Clock Cycles} \times T = \frac{\text{Clock Cycles}}{f}$$

### CPI 도입 후 확장

$$\text{Clock Cycles} = \text{ Instructions} \times \text{CPI}$$

$$\boxed{\text{CPU Time} = \text{ Instructions} \times \text{CPI} \times T = \frac{\text{\# Instructions} \times \text{CPI}}{f}}$$

- **CPI (Cycles Per Instruction)** : 명령어 하나를 실행하는 데 평균적으로 걸리는 클럭 사이클 수

---

## 7. 성능을 결정하는 세 요소

| 변수 | 영향을 주는 요소 |
|---|---|
| **# Instructions** | 알고리즘, 프로그래밍 언어, 컴파일러, ISA |
| **CPI** | 마이크로아키텍처, 컴파일러, ISA |
| **T (= 1/f)** | 반도체 기술, 마이크로아키텍처, ISA |

> ⚠️ 세 변수는 독립적이지 않다.  
> 명령어 수를 줄이려고 복잡한 명령어를 쓰면 CPI가 올라갈 수 있고,  
> 주파수를 높이면 CPI도 커지는 경향이 있다.

---

## 8. CPI 심화 — 명령어 종류별 계산

명령어 종류마다 CPI가 다를 때:

$$\text{Clock Cycles} = \sum_{i=1}^{n} (\text{CPI}_i \times \text{Count}_i)$$

$$\text{평균 CPI} = \frac{\text{Clock Cycles}}{\text{전체 명령어 수}}$$

### 예제 — 컴파일러 비교

CPI 조건: A=1, B=2, C=3

| | Sequence 1 | Sequence 2 |
|---|---|---|
| 명령어 수 | 5개 | 6개 |
| Clock Cycles | 2×1 + 1×2 + 2×3 = **10** | 4×1 + 1×2 + 1×3 = **9** |
| 평균 CPI | 10/5 = **2.0** | 9/6 = **1.5** |
| 실행시간 | 10T | **9T** ✅ |

> 💡 명령어 수가 더 많아도 Sequence 2가 빠르다.  
> **명령어 수만 보면 안 되는 이유.**

---

## 9. CPU Time 계산 — 풀이 방식

명령어 종류별 Count가 주어진 경우, 아래처럼 직접 계산하는 방식이 더 정확하다.

```
CPU Time = (Count_A × CPI_A + Count_B × CPI_B + ...) / f
```

**예시** — f = 1GHz, 총 1억 개 명령어 (ALU 50%, Load 30%, Branch 20%)

```
= (5000만×1 + 3000만×3 + 2000만×2) / 1GHz
= (5000만 + 9000만 + 4000만) / 1GHz
= 1억 8000만 / 1GHz
= 0.18초
```

---

## Summary

```
CPU Time = # Instructions × CPI / f
                ↑               ↑      ↑
           줄이려면          줄이려면  높이려면
           좋은 알고리즘     좋은 HW   좋은 공정
           좋은 컴파일러     좋은 ISA
```

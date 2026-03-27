# [TIL] 프로그래밍 언어론: 공리적 의미론 (Axiomatic Semantics) 정리

## 1. 공리적 의미론이란?
코드를 컴퓨터에 직접 실행해 보지 않고, 수학적 논리와 절대 규칙(공리)만을 사용하여 **"이 프로그램이 오류 없이 완벽하게 작동함"을 종이 위에서 증명하는 방법**이다. 우주선 제어, 금융 시스템 등 절대 실패해서는 안 되는 크리티컬한 소프트웨어의 무결성을 증명할 때 사용된다.

## 2. 핵심 개념: Assertion (단언)과 Hoare Triple
코드가 어떤 행동(Statement)을 수행하도록 지시하는 명령문이 아니라, 특정 시점에 **"이 변수의 상태는 무조건 이러하다!"라고 팩트를 선언하는 수학적 문장(논리식)**이다.   
코드가 한 줄씩 실행되는 과정을 '정지' 시켜놓고, **"지금 이 순간 이 변수의 상태는 무조건 이러하다!"**라고 팩트 체크를 하는 카메라와 같다.

* **사전 조건 (Pre-condition, $P$):** 명령이 실행되기 **전**에 카메라로 찍어본 상태. 이 코드가 에러 없이 무사히 실행되고 원하는 결과를 내기 위해, 시작 시점에 반드시 참(True)이어야만 하는 최소한의 전제 조건이다.
* **사후 조건 (Post-condition, $Q$):** 명령이 실행된 **후**에 카메라로 찍어본 상태. 코드가 끝난 뒤에 반드시 이렇게 되어 있을 것이라고 100% 보장하는 결과값(목적지)이다.
* **호어 트리플 (Hoare Triple):** $\lbrace P \rbrace \ S \ \lbrace Q \rbrace$ (조건 $P$에서 명령 $S$를 실행하면 결과 $Q$가 나온다.)

---

## 3. 핵심 추론 규칙 4가지 (Inference Rules)

### ① 귀결 규칙 (Rule of Consequence) : "조건의 융통성"
* **🎯 존재 이유 (Why?):** 기계적인 공식 대입으로 뽑아낸 조건(예: $x > -1$)과, 사람이 실제로 증명하고 싶었던 논리 조건(예: $x > 3$)의 '모양'이 다를 때, 이를 논리적으로 연결해 주는 **유연한 징검다리**가 필요하기 때문이다.
* **[Step 1] 기본 팩트 확인:** 원래의 코드 $S$가 조건 $P$에서 실행되면 결과 $Q$를 보장한다는 사실을 안다. ($\lbrace P \rbrace S \lbrace Q \rbrace$)
* **[Step 2] 사전 조건 강화:** 원래 필요한 $P$보다 더 빡빡하고 확실한 조건($P^{\prime}$)을 주고 시작해도 코드는 당연히 잘 돌아간다. ($P^{\prime} \Rightarrow P$)
* **[Step 3] 사후 조건 약화:** 코드가 보장하는 확실한 결과 $Q$를 조금 더 느슨하게 뭉뚱그려 표현($Q^{\prime}$)해도 거짓말이 아니다. ($Q \Rightarrow Q^{\prime}$)
$$\frac{\lbrace P \rbrace S \lbrace Q \rbrace, \quad P^{\prime} \Rightarrow P, \quad Q \Rightarrow Q^{\prime}}{\lbrace P^{\prime} \rbrace S \lbrace Q^{\prime} \rbrace}$$

### ② 순서 규칙 (Sequence Rule) : "바통 터치"
* **🎯 존재 이유 (Why?):** 실제 프로그램은 한 줄짜리가 아니기 때문이다. 수십, 수백 줄의 코드가 끊어짐 없이 안전하게 연결되어 전체 프로그램이 작동함을 증명하기 위한 **수학적 접착제** 역할을 한다.
* **[Step 1] 1번 주자 증명:** 첫 번째 코드 $S1$을 실행하면 중간 지점인 $P2$에 무사히 도착함을 증명한다. ($\lbrace P1 \rbrace S1 \lbrace P2 \rbrace$)
* **[Step 2] 2번 주자 증명:** 두 번째 코드 $S2$가 그 중간 지점 $P2$에서 바통을 이어받아 최종 목적지 $P3$에 도착함을 증명한다. ($\lbrace P2 \rbrace S2 \lbrace P3 \rbrace$)
* **결론:** 중간 다리($P2$)가 완벽히 연결되었으므로, 전체 코드($S1; S2$)는 시작점 $P1$에서 최종점 $P3$까지 안전하게 도달한다.
$$\frac{\lbrace P1 \rbrace S1 \lbrace P2 \rbrace, \quad \lbrace P2 \rbrace S2 \lbrace P3 \rbrace}{\lbrace P1 \rbrace S1; S2 \lbrace P3 \rbrace}$$

### ③ 조건문 규칙 (Selection / If Rule) : "갈림길"
* **🎯 존재 이유 (Why?):** 프로그램은 상황에 따라 수많은 갈림길(Branch)을 타게 되는데, **사용자가 어떤 최악의 선택을 하더라도 프로그램이 뻗지 않고 예상 가능한 안전한 상태($Q$)로 합류함**을 보장하기 위해 필요하다.
* **[Step 1] 왼쪽 길 (True):** 문지기의 질문 $B$가 참일 때($B \text{ and } P$), $S1$ 길로 걸어가도 목적지 $Q$에 도착해야 한다.
* **[Step 2] 오른쪽 길 (False):** 문지기의 질문 $B$가 거짓일 때($(\text{not } B) \text{ and } P$), $S2$ 길로 걸어가도 똑같은 목적지 $Q$에 도착해야 한다.
* **결론:** 두 길 모두 $Q$로 향하므로, 이 `if`문 전체는 항상 $Q$를 보장한다.
$$\frac{\lbrace B \text{ and } P \rbrace S1 \lbrace Q \rbrace, \quad \lbrace (\text{not } B) \text{ and } P \rbrace S2 \lbrace Q \rbrace}{\lbrace P \rbrace \text{ if } B \text{ then } S1 \text{ else } S2 \lbrace Q \rbrace}$$

### ④ 반복문 규칙 (While Rule)과 루프 불변성 (Loop Invariant)
* **🎯 존재 이유 (Why?):** 코드가 10번 돌지, 1만 번 돌지 실행 전에는 알 수 없는 상황에서, 코드를 한 줄씩 펼쳐서 증명하는 것은 불가능하다. 따라서 반복 횟수에 상관없이 **영원히 변하지 않는 절대 규칙(불변성) 하나로 묶어두어, 무한 루프나 오류에 빠지지 않음을 통제**하기 위해 발명되었다.
* **[Step 1] 마법의 밧줄($I$) 찾기:** 루프가 돌기 전, 도는 중간, 끝난 후에도 절대 변하지 않고 항상 참(True)을 유지하는 '루프 불변성($I$)' 조건을 찾아낸다.
* **[Step 2] 쳇바퀴 증명:** 조건이 맞아($B$) 루프 안으로 들어갔을 때, 코드 $S$를 한 바퀴 실행하고 나면 여전히 불변성($I$)이 참인지 확인한다. ($\lbrace I \text{ and } B \rbrace S \lbrace I \rbrace$)
* **결론:** 루프가 완전히 종료되면, 불변성은 살아남고($I$), 루프 조건은 거짓이 되어 끝났음($\neg B$)을 확신할 수 있다. 이 두 팩트($I \land \neg B$)를 합쳐서 최종 사후 조건을 도출해 낸다.
$$\frac{\lbrace I \text{ and } B \rbrace S \lbrace I \rbrace}{\lbrace I \rbrace \text{ while } B \text{ do } S \lbrace I \text{ and } (\text{not } B) \rbrace}$$

---

## 4. (부록) 4가지 Semantics(의미론) 비교 요약
컴파일러와 언어 설계자들은 다음 4가지 의미론을 섞어 써서 완벽한 프로그래밍 언어를 만든다.
1.  **정적 의미론 (Static):** 실행 전 타입 검사 (예: 정수와 불리언의 합산 오류 잡기)
2.  **동작적 의미론 (Operational):** 컴퓨터 메모리 상태가 단계별(Step)로 변하는 과정 설명
3.  **표시적 의미론 (Denotational):** 프로그램 코드를 엄밀한 수학적 함수 $f(x)$로 매핑
4.  **공리적 의미론 (Axiomatic):** Assertion을 이용해 코드의 무결성을 수학 논리로 증명

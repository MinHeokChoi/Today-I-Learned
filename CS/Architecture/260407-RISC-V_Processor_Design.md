# [TIL] Computer Architecture: Single-Cycle RISC-V Processor Design

## 1. Microarchitecture 개요
* **정의**: ISA(명령어 집합 구조)를 실제로 실행하기 위한 CPU 내부의 하드웨어 구조를 의미함.
* **Single-Cycle 방식**: 모든 명령어를 단 하나의 클럭 사이클(Single cycle) 내에 실행하는 방식임.
  * **장점**: 설계가 단순하고 CPI(Cycles Per Instruction)가 1임.
  * **단점**: 가장 느린 명령어(보통 lw)의 지연 시간이 전체 클럭 주기를 결정하므로, 클럭 속도(Frequency)를 높이기 어려움.

## 2. 핵심 구성 요소 (Datapath)
CPU는 크게 데이터가 흐르는 **Datapath**와 이를 통제하는 **Control Unit**으로 나뉨.
* **PC (Program Counter)**: 다음에 실행할 명령어의 주소를 저장하는 32비트 레지스터임.
* **Instruction Memory**: PC가 가리키는 주소에서 32비트 명령어를 읽어옴.
* **Register File**: 데이터를 읽고 쓰는 저장소로, 동시에 2개를 읽고 1개를 쓸 수 있음 (3-ported).
* **ALU (Arithmetic Logic Unit)**: 실제 산술/논리 연산이나 주소 계산을 수행함.
* **Data Memory**: lw(로드)나 sw(스토어) 시 데이터를 읽거나 쓰는 외부 저장소임.


## 3. 명령어 실행 5단계 (Execution Steps)
모든 RISC-V 명령어는 하드웨어 내에서 다음의 공통 흐름을 가짐:
1. **Instruction Fetch**: PC 주소의 명령어를 메모리에서 가져옴.
2. **Instruction Decode**: 명령어를 필드별(Opcode, rs1, rs2, rd 등)로 분해하고 제어 신호를 생성함.
3. **Execute**: ALU를 통해 연산을 수행하거나 주소를 계산함.
4. **Memory Access**: 필요한 경우 데이터 메모리에 접근함 (lw, sw 전용).
5. **Write-back**: 최종 결과를 레지스터 파일에 저장함.

## 4. 핵심 제어 신호 (Control Signals)
Control Unit은 명령어의 Opcode를 분석하여 각 부품에 지시를 내림.

| 신호명 | 역할 | 0일 때 | 1일 때 |
| :--- | :--- | :--- | :--- |
| **ALUSrc** | ALU의 두 번째 입력 결정 | 레지스터 값(rs2) 사용 | 상수(Immediate) 사용 |
| **MemtoReg** | 레지스터에 쓸 데이터 선택 | ALU 계산 결과값 | 메모리에서 읽은 값 |
| **RegWrite** | 레지스터 저장 여부 | 저장 안 함 | 저장함 |
| **PCSrc** | 다음 실행 주소 결정 | PC + 4 (다음 줄) | Branch Target (분기 주소) |

> **PCSrc 결정 논리**: Branch 신호(명령어 의도)와 Zero 플래그(ALU 비교 결과)가 모두 1일 때만 1이 됨.

## 5. 특수 명령어 처리: JAL (Jump and Link)
* **목적**: 무조건 점프를 수행하고 돌아올 주소를 저장함.
* **동작**:
  * PC는 현재 PC에 상수를 더한 **Jump Target**으로 바뀜.
  * 나중에 돌아올 주소인 **PC + 4**를 레지스터(rd)에 저장함 (Link 과정).

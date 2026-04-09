# [Computer Architecture] RISC-V Control Unit 설계: ALUOp와 ALUControl의 차이

## 1. ALUOp vs ALUControl: 왜 같은 `00`인데 연산이 다를까?

결론부터 말하자면, 두 신호는 **서로 다른 계층(Layer)**에서 작동하는 별개의 신호이기 때문이다.

### (1) Main Decoder의 ALUOp (2-bit)
메인 디코더는 명령어의 `opcode`만 보고 "이 명령어가 어떤 범주의 연산을 수행하는가"를 결정하여 **ALUOp** 신호를 만든다.
- **00**: "무조건 **더하기(Add)** 해라" (주소 계산이 필요한 `lw`, `sw` 등)
- **01**: "무조건 **빼기(Subtract)** 해라" (비교가 필요한 `beq` 등)
- **10**: "나도 모르니 `funct` 필드를 보고 니가 정해라" (R-type 연산)

### (2) ALU Decoder의 ALUControl (3-bit)
ALU 디코더는 메인 디코더가 준 `ALUOp`와 명령어의 세부 필드(`funct3`, `funct7`)를 조합해 실제 ALU를 물리적으로 움직이는 **ALUControl** 신호를 생성한다.
- **010**: 실제 ALU 내부에서 **Add** 연산을 수행하는 신호
- **110**: 실제 ALU 내부에서 **Subtract** 연산을 수행하는 신호
- **000**: 실제 ALU 내부에서 **AND** 연산을 수행하는 신호

> **정리:** `lw` 명령어의 경우, Main Decoder는 `ALUOp`로 `00`(대분류: Add)을 내보내고, 이를 받은 ALU Decoder는 ALU에게 실제 더하기를 시키기 위해 `ALUControl` 신호로 `010`을 쏴주는 것이다. 따라서 각 테이블의 `00`은 서로 의미하는 바가 다르다.

---

## 2. 왜 제어 유닛을 2단계(계층적 구조)로 설계할까?

모든 제어 신호를 `opcode` 하나만 보고 한 번에 뽑아내지 않고, Main Decoder와 ALU Decoder로 나누는 이유는 다음과 같다.

### ① 하드웨어 설계의 효율성 (Efficiency)
- 모든 경우의 수를 하나의 거대한 진리표(Truth Table)로 만들면 로직 게이트가 복잡해지고 칩 면적을 많이 차지한다.
- 계층을 나누면 각 디코더의 크기를 작게 유지할 수 있어 하드웨어가 단순해진다.

### ② 모듈화 및 확장성 (Modularity)
- 만약 새로운 산술 연산(예: 곱셈)을 추가하고 싶다면, 메인 디코더는 건드리지 않고 **ALU 디코더만 수정**하면 된다.
- 소프트웨어의 함수 분리와 비슷하게 하드웨어도 역할을 분리하여 유지보수를 쉽게 만든 것이다.

### ③ Opcode 중복 처리 (Handling Overlap)
- RISC-V의 R-type 명령어(add, sub, and, or...)는 모두 **동일한 Opcode**를 가진다.
- 따라서 Opcode만 봐서는 구체적인 연산을 알 수 없으므로, 일단 "이것은 R-type이다"라고 분류(Main)한 뒤, 세부 비트를 다시 검사(ALU Decoder)하는 과정이 논리적으로 반드시 필요하다.

---

## 💡 요약 및 핵심
- **Main Decoder**: `opcode`를 보고 큰 틀의 제어 신호와 `ALUOp`를 결정.
- **ALU Decoder**: `ALUOp`와 `funct` 비트를 조합해 최종 `ALUControl` 결정.
- **계층 구조의 장점**: 설계 단순화, 유지보수 용이, Opcode 중복 해결.

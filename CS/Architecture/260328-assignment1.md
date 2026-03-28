# RISC-V 어셈블리 기초 패턴 정리

> 컴퓨터구조 과제1 풀이를 통해 학습한 내용 정리

## 1. 기본 산술 연산 → RISC-V 변환

### C 코드

```c
f = g + (h - 5);
// f: s0, g: s1, h: s2
```

### RISC-V

```asm
addi t0, s2, -5      # t0 = h - 5
add  s0, t0, s1      # f = (h - 5) + g
```

### 포인트

- `addi`는 즉시값(상수)을 더할 때, `add`는 레지스터끼리 더할 때 사용
- 임시 계산 결과는 t 레지스터(t0~t6)에 저장

---

## 2. 배열 접근 — 주소 계산이 핵심

### C 코드

```c
B[8] = A[i - j];
// i: s3, j: s4, A base: s6, B base: s7
```

### RISC-V

```asm
sub  t0, s3, s4       # t0 = i - j (인덱스 계산)
slli t0, t0, 2        # t0 = (i - j) * 4 (바이트 오프셋 변환)
add  t1, s6, t0       # t1 = &A[i - j] (실제 메모리 주소)
lw   t2, 0(t1)        # t2 = A[i - j]
sw   t2, 32(s7)       # B[8] = A[i - j] (8 * 4 = 32)
```

### 핵심 규칙

- 32비트 정수 배열에서 **인덱스 → 바이트 오프셋** 변환 시 반드시 **×4** 필요
- `slli rd, rs, 2`로 왼쪽 2비트 시프트 = ×4 효과
- `lw`/`sw`의 오프셋은 **즉시값(상수)**만 가능, 레지스터 불가
- 상수 인덱스는 오프셋 직접 계산 가능 (예: `B[8]` → `32(s7)`)
- 변수 인덱스는 `slli` + `add`로 주소 계산 필요

---

## 3. RISC-V → C 역변환 (패턴 인식)

### RISC-V

```asm
slli x30, x5, 2       # x30 = f * 4
add  x30, x10, x30    # x30 = &A[f]
slli x31, x6, 2       # x31 = g * 4
add  x31, x11, x31    # x31 = &B[g]
lw   x5, 0(x30)       # x5 = A[f]
addi x12, x30, 4      # x12 = &A[f + 1]
lw   x30, 0(x12)      # x30 = A[f + 1]
add  x30, x30, x5     # x30 = A[f + 1] + A[f]
sw   x30, 0(x31)      # B[g] = A[f + 1] + A[f]
```

### C 코드

```c
B[g] = A[f + 1] + A[f];
```

### 자주 보이는 패턴

| 어셈블리 패턴 | 의미 |
|---|---|
| `slli` + `add` + `lw` | 배열 인덱스 접근 `A[i]` |
| `addi reg, reg, 4` + `lw` | 포인터 순회 방식 배열 접근 |
| `blt` / `bge` + 레이블 | 루프 또는 조건문 |
| `addi reg, reg, 1` | `i++` 카운터 증가 |
| `slli` + `add` (곱셈 트릭) | 상수 곱셈 (예: `slli 3` + `add` = ×9) |

---

## 4. 엔디안 (Endianness)

`0xabcdef12`를 주소 0부터 저장할 때:

### Little-endian (LSB가 낮은 주소)

| 주소 | 값 |
|---|---|
| 0 | 12 (LSB) |
| 1 | ef |
| 2 | cd |
| 3 | ab (MSB) |

### Big-endian (MSB가 낮은 주소)

| 주소 | 값 |
|---|---|
| 0 | ab (MSB) |
| 1 | cd |
| 2 | ef |
| 3 | 12 (LSB) |

### 포인트

- 메모리 주소는 **바이트 단위** (0, 1, 2, 3)
- `lw`의 정렬 조건: **최종 접근 주소**가 4의 배수이면 OK (오프셋 자체가 4의 배수일 필요 없음)

---

## 5. 시프트 연산

### C 코드

```c
A = C[0] << 4;
// x6 = A, x17 = C의 base address
```

### RISC-V

```asm
lw   t0, 0(x17)       # t0 = C[0]
slli x6, t0, 4        # A = C[0] << 4
```

---

## 6. 루프 → C 변환

### RISC-V

```asm
addi  x5, x0, 0       # i = 0
addi  x6, x0, 0       # result = 0
addi  x29, x0, 100    # 루프 상한 = 100
LOOP: lw   x7, 0(x10)  # x7 = MemArray[현재]
      add  x6, x6, x7  # result += MemArray[현재]
      addi x10, x10, 4 # 포인터를 다음 원소로 이동
      addi x5, x5, 1   # i++
      blt  x5, x29, LOOP  # i < 100이면 반복
```

### C 코드

```c
i = 0;
result = 0;
do {
    result += MemArray[i];
    i++;
} while (i < 100);
```

### 배열 순회 방식 비교

| 방식 | 특징 | 예시 |
|---|---|---|
| 포인터 방식 | `addi base, base, 4`로 포인터 이동 | 위 코드 (베이스 주소 변경됨) |
| 인덱스 방식 | 매 반복마다 `slli` + `add`로 주소 재계산 | 베이스 주소 보존됨 |

---

## 7. 조건문 패턴

### C 코드

```c
if (x == y)
    result = a + b;
else
    result = a - b;
```

### RISC-V

```asm
bne  x5, x6, else     # x != y이면 else로 점프
add  x29, x7, x28     # result = a + b
jal  x0, END           # else 건너뛰기 (j END)
else:
sub  x29, x7, x28     # result = a - b
END:
```

### 포인트

- 조건문은 **조건을 반전**시켜서 분기: `if (x == y)` → `bne`로 else 점프
- if 블록 끝에 **무조건 점프**(`jal x0, END` 또는 `j END`)로 else 건너뛰기 필수
- `jal x0, label`은 리턴 주소를 x0(zero)에 저장 → 사실상 버림 → 단순 점프

---

## 8. 레지스터 분류 (ABI 별명)

| 별명 | 레지스터 | 용도 |
|---|---|---|
| zero | x0 | 항상 0 |
| ra | x1 | 리턴 주소 |
| sp | x2 | 스택 포인터 |
| t0~t6 | x5-7, x28-31 | 임시 (함수 호출 시 보존 안 됨) |
| s0~s11 | x8-9, x18-27 | 저장 (함수 호출 시 보존됨) |
| a0~a7 | x10-17 | 함수 인자/리턴값 |

- x 번호는 외울 필요 없고, **별명으로 기억**하면 충분
- 시험에서는 보통 레지스터 표를 제공함

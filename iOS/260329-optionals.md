# Swift 옵셔널 (Optionals) 핵심 개념

## 옵셔널이란?

값이 **있을 수도 있고 없을 수도(`nil`) 있는** 타입

```swift
var name: String? = "민혁"   // 값이 있는 옵셔널
var age: Int? = nil          // 값이 없는 옵셔널
```

---

## 1. 강제 언래핑 `!`

옵셔널 안의 값을 **강제로 꺼낸다**. nil일 때 사용하면 **크래시**.

```swift
var city: String? = "서울"
print(city!)   // "서울"

city = nil
print(city!)   // 💥 런타임 크래시!
```

> C++ 비유: null 포인터를 체크 없이 `*ptr` 역참조하는 것과 같다. 되도록 쓰지 말자.

---

## 2. `if let` — 안전한 언래핑

nil이 아닐 때만 값을 꺼내서 사용한다.

```swift
var email: String? = "test@example.com"

if let unwrappedEmail = email {
    print("이메일 있음: \(unwrappedEmail)")
} else {
    print("이메일 없음")
}
```

> C++ 비유: `if (ptr != nullptr) { ... }`

---

## 3. `guard let` — 조기 탈출

nil이면 **함수를 바로 빠져나간다**. `if let`과 달리 언래핑한 값을 함수 전체에서 쓸 수 있다.

```swift
func greetUser(nickname: String?) {
    guard let name = nickname else {
        print("닉네임이 없어서 인사할 수 없음")
        return
    }
    // 여기서부터 name은 String (옵셔널 아님)
    print("안녕하세요, \(name)님!")
}
```

> C++ 비유: 함수 초반에 `if (!ptr) return;` 패턴

### `if let` vs `guard let` 사용 기준

| | `if let` | `guard let` |
|---|---|---|
| 언래핑 값 사용 범위 | if 블록 안에서만 | 함수 전체 |
| 주로 쓰는 곳 | 부분적으로 nil 처리할 때 | 함수 초반 유효성 검사 |

---

## 4. nil 병합 연산자 `??`

**nil이면 기본값**을 사용한다.

```swift
var theme: String? = nil
let currentTheme = theme ?? "기본 테마"
print(currentTheme)  // "기본 테마"

theme = "다크 모드"
let updatedTheme = theme ?? "기본 테마"
print(updatedTheme)  // "다크 모드"
```

> C++ 비유: `theme != nullptr ? *theme : "기본 테마"` (삼항 연산자의 nil 체크 전용 축약형)

---

## 5. 옵셔널 체이닝 `?.`

nil이 아니면 뒤의 속성/메서드를 실행하고, **nil이면 전체가 nil**.

```swift
var scores: [Int]? = [85, 92, 78]
print(scores?.first)  // Optional(85)

scores = nil
print(scores?.first)  // nil (체이닝에서 끊김)
```

---

## 6. `?.`와 `??` 조합

```swift
let firstScore = scores?.first ?? 0
```

| 상황 | `scores?.first` 결과 | 최종값 |
|---|---|---|
| `scores = [85, 92, 78]` | `Optional(85)` | **85** |
| `scores = []` (빈 배열) | `nil` (first가 nil) | **0** |
| `scores = nil` | `nil` (체이닝이 nil) | **0** |

**nil이 되는 이유는 다르지만 결과는 같다** → `??`가 기본값을 넣어준다.

---

## 7. 옵셔널을 문자열에 직접 넣으면 생기는 일

```swift
print("\(scores?.first)")  // ⚠️ Xcode 경고 발생
```

옵셔널을 `\()` 안에 넣으면 `Optional(85)`처럼 출력되어 Xcode가 경고를 띄운다.

**해결법:**
```swift
print("\(scores?.first ?? 0)")              // 기본값 제공
print("\(String(describing: scores?.first))") // 명시적으로 옵셔널 출력
```

---

## C++ 대응 요약

| Swift | C++ | 의미 |
|---|---|---|
| `String?` | `std::optional<std::string>` | 값이 있거나 nil |
| `!` | `*ptr` (null 체크 없이) | 강제 언래핑 — 위험 |
| `if let` | `if (ptr != nullptr)` | nil이 아닐 때만 사용 |
| `guard let` | `if (!ptr) return;` | nil이면 조기 탈출 |
| `??` | `ptr ? *ptr : default` | nil이면 기본값 |
| `?.` | `ptr ? ptr->method() : nullopt` | nil이면 전체가 nil |

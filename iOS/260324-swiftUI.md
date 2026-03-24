# 📱 SwiftUI 기초

## Extension — 남의 타입에 내 기능 추가하기

### extension이란?

Apple이 만든 `Color` 타입에는 `.red`, `.blue` 같은 기본 색상만 있다. `.megaboxPurple` 같은 커스텀 색상은 당연히 없다.

`extension`은 **이미 존재하는 타입에 새 기능을 추가**하는 문법이다. 원본 코드를 수정하지 않고 기능을 확장할 수 있다.

```swift
extension Color {
    static let megaboxPurple = Color(red: 108/255, green: 63/255, blue: 191/255)
}

// 사용: 마치 원래 있던 것처럼
Color.megaboxPurple
```

### static이란?

`static`은 **인스턴스를 만들지 않고 타입 자체에서 직접 접근**할 수 있게 해주는 키워드다.

```swift
Color.megaboxPurple  // ✅ static: 타입에서 바로 접근
// vs
let myColor = Color(...)  // 인스턴스를 만들어야 접근
```

---

## Model 설계 — struct, Identifiable, 더미데이터

### struct — 데이터를 묶는 설계도

서로 관련된 데이터를 하나의 타입으로 묶는다. class와 달리 **값 타입(value type)**이다.

```swift
struct Movie: Identifiable {
    let id = UUID()
    let title: String
    let posterImageName: String
    let ageRating: Int
    let releaseDate: String
}
```

### Identifiable 프로토콜

`Identifiable`을 채택하면 `id` 속성이 필수다. SwiftUI의 `ForEach`가 **각 아이템을 고유하게 구별**하기 위해 필요하다.

```swift
// Identifiable 채택 시: 자동으로 id로 구별
ForEach(movies) { movie in
    MoviePosterCard(movie: movie)
}

// 미채택 시: 수동으로 id 지정 필요
ForEach(movies, id: \.title) { movie in ... }
```

### UUID

`UUID()` — 호출할 때마다 전 세계에서 유일한 ID를 자동 생성한다.

### 멤버와이즈 이니셜라이저

struct는 **모든 속성을 매개변수로 받는 init을 자동 생성**한다. class에는 없는 편의 기능.

```swift
Movie(title: "F1: 더 무비", posterImageName: "poster_f1", ageRating: 12, releaseDate: "25.06.25")
// init을 따로 작성하지 않아도 됨
```

### 더미데이터와 extension 분리

**더미데이터** = 서버 연동 전에 UI를 테스트하기 위한 가짜 데이터.

extension으로 분리하면 **모델 정의(영구적)와 테스트 데이터(임시적)를 구분**할 수 있다. 나중에 서버 연동 시 extension 블록만 삭제하면 된다.

```swift
// 영구적: 모델 정의
struct Movie: Identifiable { ... }

// 임시적: 나중에 서버 연동 시 이 블록만 삭제
extension Movie {
    static let sampleMovies: [Movie] = [...]
}
```

---

## 상태 관리, Modifier, 컴포넌트 분리

### @State vs @Binding

| | @State | @Binding |
|---|--------|---------|
| 역할 | **값을 직접 소유** | **부모의 값을 빌려 씀** |
| 초깃값 | 필요 (`= ""`) | 필요 없음 |
| 관례 | `private` 붙임 | `private` 안 붙임 |

```swift
// 부모 (MegaboxApp)
@State private var isLoggedIn = false
LoginView(isLoggedIn: $isLoggedIn)  // $ 붙여서 Binding으로 전달

// 자식 (LoginView)
@Binding var isLoggedIn: Bool       // 부모의 값에 연결됨
```

### $ (달러 사인) — Binding 생성

`$`는 `@State`든 `@Binding`이든 **"값을 읽고 쓸 수 있는 안전한 채널"**을 만든다.

```swift
@State private var userId: String = ""

userId    // → String 값 (읽기 전용 복사본)
$userId   // → Binding<String> (원본에 읽기 + 쓰기 가능)
```

```swift
// TextField는 사용자 입력을 받아 원본 값을 바꿔야 하므로 Binding이 필요
TextField("아이디", text: $userId)  // ✅ 원본에 연결
TextField("아이디", text: userId)   // ❌ 복사본이라 쓰기 불가
```

### Modifier 체이닝 — 순서가 중요하다

각 modifier는 **현재 View를 감싸는 새 View를 반환**한다.

```swift
// ✅ frame 먼저 → 넓어진 뒤에 배경색
.frame(maxWidth: .infinity)   // 가로 꽉 채운 View 생성
.background(Color.purple)     // 그 넓은 View에 배경색

// ❌ background 먼저 → 글자 크기만큼만 배경색
.background(Color.purple)     // 글자 크기 View에 배경색 (이미 작음)
.frame(maxWidth: .infinity)   // 바깥을 넓혀도 배경은 안쪽에 고정
```

### Optional (?) 과 if let

`?`는 **"값이 있을 수도, 없을(nil) 수도 있다"**는 타입이다.

```swift
var label: String? = nil      // 값이 없음
var label: String? = "N"      // 값이 있음

// if let: Optional을 안전하게 꺼내기
if let label = label {
    Text(label)       // 값이 있을 때만 실행
}
```

### 컴포넌트 분리 — 반복 UI를 struct로 추출

비슷한 UI가 반복될 때, **달라지는 부분만 매개변수로 받는 별도의 View struct**를 만든다.

```swift
// 3개 소셜 버튼이 모양은 같고 색/아이콘만 다름 → 하나의 컴포넌트로
SocialLoginButton(backgroundColor: .green,  label: "N",  labelColor: .white)
SocialLoginButton(backgroundColor: .yellow, isKakao: true, ...)
SocialLoginButton(backgroundColor: .black,  systemIcon: "apple.logo", ...)
```

### `:` 구분법 — 타입 vs 매개변수 레이블

```swift
// var/let 뒤 → 타입
var userId: String

// 함수 호출 () 안 → 매개변수 레이블: 값
TextField("아이디", text: $userId)
```


# 📱 SwiftUI

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



# 📱 SwiftUI 컴포넌트 설계  — ProfileHeaderView + QuickActionButton

## Stateless 컴포넌트

`@State`나 `@Binding` 없이 **항상 같은 모습을 보여주는 View**를 Stateless 컴포넌트라 한다.

```swift
struct ProfileHeaderView: View {
    // 상태 변수 없음! 항상 동일한 화면
    var body: some View { ... }
}
```

사용자 입력을 받지 않고 변하는 값도 없는 순수한 UI 컴포넌트.

---

## HStack alignment — 세로 정렬 기준

HStack 안의 요소들 높이가 다를 때 세로 정렬 기준을 정한다.

```swift
HStack(alignment: .top) { ... }     // 윗선 맞춤
HStack(alignment: .center) { ... }  // 가운데 맞춤 (기본값)
HStack(alignment: .bottom) { ... }  // 밑선 맞춤
```

프로필 사진(56pt)과 텍스트 블록의 높이가 다르므로, `.top`으로 윗선을 맞춰 깔끔하게 정렬.

---

## Image modifier 체이닝

```swift
Image("profile_image")
    .resizable()                        // 크기 조절 가능하게 (필수!)
    .aspectRatio(contentMode: .fill)    // 비율 유지하며 채우기
    .frame(width: 56, height: 56)       // 56x56 크기 지정
    .clipShape(Circle())                // 원형으로 자르기
```

- **`.resizable()`** — 없으면 `.frame()`을 줘도 크기가 안 바뀜. 이미지 크기 조절의 필수 전제조건.
- **`.clipShape(Circle())`** — 사각형 이미지를 완벽한 원형으로 잘라냄.

### .fill vs .fit

```swift
.aspectRatio(contentMode: .fill)  // 꽉 채움. 넘치는 부분 잘림 → 프로필 사진
.aspectRatio(contentMode: .fit)   // 안에 맞춤. 잘리지 않음    → 아이콘
```

프로필 사진은 잘려도 괜찮지만, 아이콘은 전체가 보여야 하므로 용도에 따라 선택.

---

## Spacer()로 양쪽 밀어내기

```swift
HStack {
    VStack { /* 이름+포인트 */ }
    Spacer()           // ← 남은 공간을 전부 차지
    Button("회원정보")  // ← 오른쪽 끝으로 밀림
}
```

HStack 안에서 Spacer()는 남은 가로 공간을 먹어서 좌우 요소를 양쪽 끝으로 밀어낸다.

---

## padding 방향 지정

```swift
.padding()                  // 상하좌우 전부 (기본 16pt)
.padding(20)                // 상하좌우 전부 20pt
.padding(.horizontal, 10)  // 좌우만 10pt
.padding(.vertical, 4)     // 상하만 4pt
.padding(.top, 16)          // 위만 16pt
```

WELCOME 뱃지에서 `.padding(.horizontal, 10).padding(.vertical, 4)` → 글자 주변에 여유 공간을 만들어 뱃지 느낌을 줌.

---

## 좋은 컴포넌트 설계 원칙

> **변하는 것만 매개변수로, 안 변하는 것은 내부에 고정**

```swift
struct QuickActionButton: View {
    let imageName: String  // 변하는 것: 아이콘
    let title: String      // 변하는 것: 텍스트
    // 크기(44x44), 폰트(12), 간격(8) → 내부 고정
}
```

SocialLoginButton(매개변수 6개)보다 QuickActionButton(매개변수 2개)이 더 단순하다. 차이가 적을수록 매개변수도 적게.

---

## .frame(maxWidth: .infinity)로 균등 배치

```swift
HStack {
    QuickActionButton(...)  // 각각 .infinity로 최대한 늘어남
    QuickActionButton(...)  // → HStack 안에서 균등하게 4등분
    QuickActionButton(...)
    QuickActionButton(...)
}
```

여러 개가 HStack 안에서 동시에 `.infinity`면, 사용 가능한 공간을 **균등 분배**한다.

---

## Preview 팁 — 실제 사용 맥락에서 테스트

```swift
#Preview {
    HStack {
        QuickActionButton(imageName: "icon_movie_booking", title: "영화별예매")
        QuickActionButton(imageName: "icon_theater_booking", title: "극장별예매")
    }
}
```

컴포넌트 단독이 아니라, **실제 사용될 배치(HStack 안에 여러 개)**로 Preview하면 실전 모습을 바로 확인할 수 있다.



# TIL: MegaBox 프로젝트 MVVM 리팩토링 & SwiftUI 심화

> **학습 날짜:** 2026-04-02

---

## 학습 목표

- [x] MVVM 패턴 위반 사례 식별 및 수정
- [x] View에서 비즈니스 로직 분리하기
- [x] `@AppStorage` vs `UserDefaults` 차이 이해
- [x] computed property (get/set) 활용
- [x] enum으로 타입 안전한 상태 관리
- [x] 헬퍼 함수로 중복 코드 제거
- [x] SwiftUI 핵심 개념 총정리 (데이터 흐름, 네비게이션, modifier)

---

## 1. MVVM 리팩토링: 무엇을 왜 바꿨나

### 문제 1: View에 비즈니스 로직이 있었다

```swift
// ❌ Before — LoginView가 직접 저장
Button(action: {
    id = loginVM.loginModel.id      // View가 AppStorage에 쓰기
    pwd = loginVM.loginModel.pwd    // View가 AppStorage에 쓰기
    isLoggedIn = true               // View가 상태 변경
})

// ✅ After — View는 호출만, 로직은 ViewModel에
Button(action: {
    authVM.login()                  // "로그인 해줘" 한마디
})
```

**원칙:** View는 "뭘 할지" 결정하고, ViewModel은 "어떻게 할지" 실행한다.

### 문제 2: 로그인/로그아웃이 각각 다른 곳에 있었다

```swift
// ❌ Before — 로그인은 LoginView, 로그아웃은 LogoutButton에서 직접 처리
// 같은 "인증" 책임인데 코드가 분산됨

// ✅ After — AuthViewModel에 통합
@Observable
class AuthViewModel {
    func login() { ... }                    // 로그인
    func logout(container: DIContainer) { ... }  // 로그아웃
}
```

---

## 2. `@AppStorage` vs `UserDefaults`

```swift
// View에서는 @AppStorage 사용 가능
struct RootView: View {
    @AppStorage("isLoggedIn") private var isLoggedIn: Bool = false
}

// ViewModel(@Observable class)에서는 UserDefaults 직접 사용
@Observable
class AuthViewModel {
    var isLoggedIn: Bool {
        get { UserDefaults.standard.bool(forKey: "isLoggedIn") }
        set { UserDefaults.standard.set(newValue, forKey: "isLoggedIn") }
    }
}
```

| | @AppStorage | UserDefaults |
|---|---|---|
| **사용 가능 위치** | SwiftUI View 전용 | 어디서든 가능 |
| **저장소** | UserDefaults (동일!) | UserDefaults |
| **UI 자동 갱신** | 자동 | computed property로 대응 |

**핵심:** 둘 다 같은 UserDefaults를 읽고 쓴다. `@AppStorage("isLoggedIn")`과 `UserDefaults.standard.bool(forKey: "isLoggedIn")`은 같은 값을 가리킨다.

---

## 3. computed property (get/set)

```swift
var isLoggedIn: Bool {
    get { UserDefaults.standard.bool(forKey: "isLoggedIn") }  // 읽을 때
    set { UserDefaults.standard.set(newValue, forKey: "isLoggedIn") }  // 쓸 때
}
```

- **저장된 변수가 아니다** — 읽고 쓸 때마다 함수가 실행된다
- `newValue`는 Swift가 자동 제공하는 키워드 (대입된 값)
- C++의 getter/setter를 하나의 변수 문법으로 합친 것

```swift
authVM.isLoggedIn = true    // set 블록 실행 → UserDefaults에 저장
if authVM.isLoggedIn { }    // get 블록 실행 → UserDefaults에서 읽기
```

---

## 4. enum으로 타입 안전한 상태 관리

```swift
// ❌ Before — Int로 상태 관리 (의미 불명확)
@State private var selectedSegment: Int = 0
viewModel.currentMovies(for: selectedSegment)
// 0이 뭐지? 6개월 뒤에 기억 못함

// ✅ After — enum으로 명확한 의미 부여
enum MovieChartType {
    case nowPlaying   // 현재 상영
    case upcoming     // 상영 예정
}

@State private var selectedSegment: MovieChartType = .nowPlaying
viewModel.currentMovies(for: selectedSegment)
// 바로 이해됨!
```

**장점:**
- 오타 불가능 (`.nowPlaying` vs `0` 잘못 쓸 일 없음)
- 새 케이스 추가 시 `switch`에서 컴파일 에러로 빠짐 없이 처리

---

## 5. 헬퍼 함수 (Helper Function)

**"도와주는 함수"** — 반복되는 코드를 함수로 추출하여 재사용

```swift
// ❌ Before — 거의 동일한 코드가 두 번 (약 40줄)
Button(action: { selectedSegment = .nowPlaying }) {
    Text("무비차트")
        .font(.pretendardSemiBold14)
        .foregroundStyle(selectedSegment == .nowPlaying ? .white : Color(.gray03))
        // ... 10줄 더
}
Button(action: { selectedSegment = .upcoming }) {
    Text("상영예정")
        .font(.pretendardSemiBold14)
        .foregroundStyle(selectedSegment == .upcoming ? .white : Color(.gray03))
        // ... 같은 10줄 반복
}

// ✅ After — 헬퍼 함수로 2줄
chartToggleButton(title: "무비차트", type: .nowPlaying)
chartToggleButton(title: "상영예정", type: .upcoming)

private func chartToggleButton(title: String, type: HomeViewModel.MovieChartType) -> some View {
    // 공통 스타일 한 번만 작성
}
```

---

## 6. SwiftUI 핵심 개념 정리 (프로젝트 학습)

### @Environment 주입 체인

```
RootView (@State container, @State authVM)
├── .environment(container)   → MainTabView가 받음
├── .environment(authVM)      → LoginView / LogoutButton이 받음
│
└── MainTabView
    ├── .environment(container.homeRouter)  → HomeView가 받음
    └── .environment(container.myPageRouter) → MyPageView가 받음
```

**규칙:** `@Environment`로 받으려면, 반드시 **조상 뷰**에서 `.environment()`로 주입해야 한다. Preview에서도 마찬가지.

### Toolbar vs NavigationBar

- **NavigationBar** = 상단의 **영역(공간)** 자체
- **.toolbar { }** = 그 영역에 **아이템을 배치하는 방법**
- `.sharedBackgroundVisibility(.hidden)` = toolbar 아이템의 글래스 효과 제거
- `.toolbarBackgroundVisibility(.hidden)` = NavigationBar 배경만 투명 (영역은 유지)

### Image modifier 순서

```swift
// ❌ 에러 — .resizable() 이후에는 Image가 아닌 some View
Image("logo").resizable().renderingMode(.original)

// ✅ 정답 — Image 전용 modifier는 체인 앞쪽에
Image("logo").renderingMode(.original).resizable().scaledToFit()
```

**규칙:** `.renderingMode()`, `.resizable()`는 Image 전용. `.resizable()`이 `some View`를 반환하면 이후 Image 메서드 사용 불가.

### TabView selection ↔ value 매칭

```swift
TabView(selection: $selectedTab) {      // selectedTab = 1이면
    Tab("홈", value: 0) { HomeView() }   // 0 ≠ 1 → 안 보임
    Tab("예매", value: 1) { ReservationView() }  // 1 == 1 → 보임!
}
```

- `selection`의 값과 `value`(또는 `.tag()`)가 일치하는 탭이 표시됨
- 양방향: 탭 터치 → selection 업데이트 / selection 변경 → 탭 전환

### ScrollViewReader + proxy

```swift
ScrollViewReader { proxy in
    ScrollView(.horizontal) {
        ForEach(...) { index, item in
            view.id(index)  // 각 아이템에 id 부여
        }
    }
    .onChange(of: selectedIndex) { _, newValue in
        proxy.scrollTo(newValue, anchor: .center)  // 프로그래밍으로 스크롤
    }
}
```

- **proxy** = ScrollView를 대신 제어해주는 리모컨
- `.id()`로 각 아이템에 식별자를 부여하고, `proxy.scrollTo(id)`로 이동

### @Binding이 필요한 경우 vs 아닌 경우

```swift
// 같은 View 안 — 직접 접근 (바인딩 불필요)
@State private var selectedIndex: Int = 0
selectedIndex = 3  // ✅ 직접 쓰기

// 다른 View에게 수정 권한을 넘길 때 — $바인딩
TabView(selection: $selectedIndex)  // TabView가 값을 수정할 수 있게
NavigationStack(path: $bindableRouter.path)  // NavigationStack이 path를 수정
```

---

## 7. 리팩토링 전후 파일 변경 요약

| 파일 | 변경 내용 |
|------|----------|
| `AuthViewModel.swift` (신규) | `login()` + `logout()` 인증 로직 통합 |
| `LoginViewModel.swift` | 삭제 (AuthViewModel로 대체) |
| `LoginView.swift` | @AppStorage 제거, @Environment로 authVM 받기 |
| `RootView.swift` | AuthViewModel 생성 + environment 주입 |
| `ProfileMangaeView.swift` | LogoutButton이 AuthViewModel 사용 |
| `HomeViewModel.swift` | MovieChartType enum 추가 |
| `HomeView.swift` | enum 적용 + 토글 버튼 헬퍼 추출 |

---

## 8. DIContainer vs AuthViewModel — 왜 분리하는가?

```swift
// RootView에서 둘 다 생성하지만, 별도로 관리
@State private var container = DIContainer()
@State private var authVM = AuthViewModel()
```

**DIContainer에 AuthViewModel을 넣지 않는 이유: 책임이 다르다.**

| | DIContainer | AuthViewModel |
|---|---|---|
| **책임** | 앱 인프라 (네비게이션 경로, 탭 상태) | 인증 비즈니스 로직 (로그인/로그아웃) |
| **담긴 것** | homeRouter, myPageRouter, selectedTab | loginModel, login(), logout() |
| **비유** | 건물의 배관/전기 시스템 | 건물의 경비 팀 |

```swift
// ❌ DIContainer에 넣으면 — Container가 "인증"까지 알아야 함 (책임 과다)
class DIContainer {
    var homeRouter = ...
    var authVM = AuthViewModel()  // 인프라 상자에 인증 로직?
}

// ✅ 분리하면 — 각자 자기 역할만
class DIContainer { homeRouter, myPageRouter, selectedTab }  // 인프라
class AuthViewModel { login(), logout() }                     // 인증
```

**확장성:** 나중에 API 서버 연동 시 `AuthService`, `TokenManager` 같은 것은 AuthViewModel 안에서, `NetworkService`, `ImageCache` 같은 것은 DIContainer에 추가하면 된다.

---

## 핵심 교훈

1. **View는 UI만, 로직은 ViewModel에** — 가장 중요한 MVVM 원칙
2. **같은 책임은 하나의 ViewModel에 모은다** — 불필요한 ViewModel 생성 방지
3. **다른 책임은 분리한다** — DIContainer(인프라)와 AuthViewModel(인증)을 섞지 않는다
4. **Int보다 enum** — 타입 안전성 + 가독성
5. **중복 코드는 헬퍼로 추출** — 변경 시 한 곳만 수정
6. **프레임워크의 약속(Convention)은 학습해야 한다** — selection↔value, @Environment 주입 등은 코드만 봐서 유추 불가

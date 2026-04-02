# 🗂️ MegaBox 프로젝트 Week1~Week3에서 배운 SwiftUI 핵심 요소

## 1. 앱 구조 & 생명주기

> SwiftUI 앱이 어디서 시작되고 어떻게 첫 화면을 결정하는지를 이해해야, 로그인 분기·온보딩 같은 **앱 진입 흐름**을 자유롭게 설계할 수 있다.

| 개념 | 사용 위치 | 역할 |
|------|----------|------|
| `@main` + `App` 프로토콜 | UMCMegaBoxApp.swift | 앱 진입점 |
| `WindowGroup` | UMCMegaBoxApp.swift | 창 관리 |
| `@AppStorage` | RootView, LoginView, ProfileManageView | UserDefaults 영구 저장 |

```swift
// 앱 시작 → RootView → isLoggedIn에 따라 분기
@main
struct UMCMegaBoxApp: App {
    var body: some Scene {
        WindowGroup { RootView() }
    }
}

// 조건부 화면 전환
struct RootView: View {
    @AppStorage("isLoggedIn") private var isLoggedIn: Bool = false
    if isLoggedIn { MainTabView() } else { LoginView() }
}
```

---

## 2. 데이터 흐름 (Property Wrappers)

> SwiftUI는 데이터가 변하면 UI가 자동으로 갱신되는 구조다. 어떤 Wrapper를 써야 데이터가 올바른 범위에서 흐르는지 모르면 **UI가 안 바뀌거나 예상 밖의 화면이 리렌더링**되는 버그가 생긴다.

| Wrapper | 사용 위치 | 역할 |
|---------|----------|------|
| `@State` | HomeView, MovieDetailView | View가 소유하는 로컬 상태 |
| `@Bindable` | HomeView, MainTabView, LoginView | `@Observable` 객체를 `$`바인딩으로 쓸 때 |
| `@Environment` | HomeView, MainTabView, LogoutButton | 부모에서 주입받은 객체 접근 |
| `@AppStorage` | RootView, LoginView, MemberInfo | UserDefaults와 양방향 바인딩 |
| `@Observable` | DIContainer, HomeViewModel, LoginViewModel, NavigationRouter | 관찰 가능한 클래스 |

```swift
// 흐름 요약
@Observable class → @Environment로 주입 → @Bindable로 $바인딩
@State            → 같은 View에서 직접 읽기/쓰기
@AppStorage       → 앱 종료 후에도 유지되는 값
```

---

## 3. 네비게이션 구조

> 화면이 많아질수록 네비게이션 로직이 View 곳곳에 흩어지면 관리가 불가능해진다. **NavigationStack + Router 패턴**으로 경로를 한 곳에서 제어하면, 딥링크·로그아웃 초기화 등 복잡한 흐름도 깔끔하게 처리할 수 있다.

| 개념 | 사용 위치 | 역할 |
|------|----------|------|
| `NavigationStack` + `path` | HomeView, MyPageView | 프로그래밍 방식 네비게이션 |
| `NavigationPath` | NavigationRouter | 타입 소거된 경로 스택 |
| `.navigationDestination(for:)` | HomeView, MyPageView | 라우트 → 화면 매칭 |
| `@Environment(\.dismiss)` | MovieDetailView | 이전 화면으로 돌아가기 |
| `NavigationRouter<T>` | 제네릭 라우터 | push/pop/reset 중앙 관리 |

```swift
// 전체 네비게이션 흐름
NavigationStack(path: $router.path)
    → router.push(.movieDetail(movie))     // 화면 추가
    → .navigationDestination(for:) { }     // 라우트별 화면 결정
    → dismiss() 또는 router.pop()           // 뒤로가기
    → router.reset()                        // 전체 초기화
```

---

## 4. TabView

> 대부분의 상용 앱은 하단 탭 바를 사용한다. TabView 하나로 **탭 네비게이션과 페이지 스와이프**를 모두 구현할 수 있으므로, selection-value 매칭 원리를 알면 다양한 UI 패턴에 재활용할 수 있다.

| 개념 | 사용 위치 |
|------|----------|
| `TabView(selection:)` + `Tab(value:)` | MainTabView — 4개 탭 |
| `.tabViewStyle(.page)` | HomeView — 특별관 카드 스와이프 |
| `.tag()` | 특별관 카드 — selection과 매칭 |
| `indexDisplayMode` | 페이지 인디케이터 표시 제어 |

```swift
// 탭바: selection ↔ value 매칭
TabView(selection: $selectedTab) {
    Tab("홈", value: 0) { ... }
}

// 페이지 스와이프: selection ↔ tag 매칭
TabView(selection: $selectedTheaterIndex) {
    ForEach(...) { Image(...).tag(index) }
}
.tabViewStyle(.page(indexDisplayMode: .always))
```

---

## 5. 레이아웃

> SwiftUI 레이아웃은 **Stack 조합이 기본**이지만, 성능(Lazy)·반응형 크기(GeometryReader)·프로그래밍 스크롤(ScrollViewReader) 같은 도구도 적절하게 쓸 수 있어야 한다.

| 개념 | 사용 위치 | 역할 |
|------|----------|------|
| `VStack`, `HStack` | 전체 | 수직/수평 배치 |
| `ScrollView` | HomeView (영화, 로고) | 스크롤 가능 영역 |
| `LazyHStack` | 영화 카드, 로고 스크롤 | 지연 로딩 수평 배치 |
| `GeometryReader` | 특별관 카드, 세그먼트 인디케이터 | 부모 크기 읽기 |
| `ScrollViewReader` | 특별관 로고 | 프로그래밍 스크롤 이동 |
| `ZStack` | 특별관 카드 | 이미지 위 텍스트 오버레이 |
| `Spacer` | 여러 곳 | 남은 공간 채우기 |

---

## 6. 이미지 & 스타일링

> 같은 이미지라도 `renderingMode`, `clipShape`, `LinearGradient` 조합에 따라 **완전히 다른 느낌**이 된다. 

| 개념 | 사용 위치 |
|------|----------|
| `.renderingMode(.original)` | Toolbar 로고 |
| `.renderingMode(.template)` | 특별관 로고 tint |
| `.resizable()` + `.scaledToFit/Fill` | 모든 이미지 |
| `.clipShape()`, `.clipped()` | 이미지 잘라내기 |
| `.cornerRadius()` | 카드, 버튼 모서리 |
| `LinearGradient` | 클럽 멤버십, 특별관 카드 오버레이, 뉴모피즘 테두리 |
| `.shadow()` + `.shadow(.inner(...))` | 뉴모피즘 효과 |
| `.overlay()` | 테두리, 인디케이터 |

---

## 7. Toolbar

> 네비게이션 바의 버튼·타이틀·배경을 세밀하게 커스터마이징하려면 Toolbar API를 알아야 한다. 기본 뒤로가기 버튼을 숨기고 **브랜드에 맞는 헤더**를 만드는 것이 중요하다.

| 개념 | 사용 위치 |
|------|----------|
| `.toolbar { ToolbarItem(placement:) }` | HomeView, MovieDetailView |
| `.sharedBackgroundVisibility(.hidden)` | 글래스 효과 제거 |
| `.navigationBarBackButtonHidden(true)` | 커스텀 뒤로가기 버튼 |
| `.navigationBarTitleDisplayMode(.inline)` | 타이틀 작게 표시 |
| `.safeAreaBar(edge: .top)` | 헤더 세그먼트 고정 |

---

## 8. 의존성 주입 (DI) 패턴

> View가 직접 ViewModel이나 Router를 생성하면 **테스트·교체가 어렵고 결합도가 높아진다**. `.environment()`로 외부에서 주입하면 View는 "무엇을 받아 쓸지"만 알면 되므로, 구조가 유연해지고 로그아웃 시 상태 초기화 같은 전역 동작도 한 곳에서 처리할 수 있다.

```
App
└── RootView (@AppStorage로 로그인 분기)
    └── MainTabView (.environment(container))
        ├── HomeView (.environment(homeRouter), .environment(container))
        │   └── MovieDetailView (NavigationStack push)
        └── MyPageView (.environment(myPageRouter), .environment(container))
            └── ProfileManageView (.environment(container) 상속)
```

**DIContainer가 중앙에서 관리하는 것들:**

- `selectedTab` — 탭 전환
- `homeRouter` — 홈 네비게이션
- `myPageRouter` — 마이페이지 네비게이션
- `resetAll()` — 로그아웃 시 전체 초기화

---

## 9. MVVM 패턴

> View에 비즈니스 로직이 섞이면 코드가 금방 비대해진다. **Model-ViewModel-View**로 역할을 분리하면 UI 변경이 로직에 영향을 주지 않고, AI가 생성한 코드를 리뷰할 때도 "이 코드가 어느 레이어에 있어야 하는지" 판단 기준이 명확해진다.

```
Model (데이터 구조)
├── MovieModel      — Identifiable, Hashable, 포맷팅 computed property
├── TheaterModel    — 특별관 정보
└── LoginModel      — 로그인 자격증명

ViewModel (비즈니스 로직)
├── HomeViewModel   — 영화 목록, 극장 목록, 세그먼트 필터링
└── LoginViewModel  — 로그인 상태 래핑

View (UI만 담당)
├── HomeView        — viewModel.currentMovies(for:) 호출
├── LoginView       — loginVM.loginModel 바인딩
└── MovieDetailView — movie 프로퍼티로 데이터 표시
```

---

## 10. 재사용 컴포넌트

> 비슷한 UI를 복붙하면 수정 시 전부 찾아 고쳐야 한다. **파라미터만 바꿔 재사용**할 수 있는 컴포넌트로 추출하면 유지보수 비용이 크게 줄고, 디자인 일관성도 자연스럽게 지켜진다.

| 컴포넌트 | 파일 | 재사용 방법 |
|---------|------|------------|
| `ProfileHeaderView` | Components/ | 이름, 포인트, 회원정보 버튼 |
| `QuickActionButton` | Components/ | imageName만 바꿔서 4개 버튼 생성 |
| `StatItem` | MyPageView 내부 | title/value만 바꿔서 3개 항목 |
| `infoRow(label:value:)` | MovieDetailView | 6개 영화 정보 행 |

---

## 11. Font 확장 (Extension)

> 폰트를 매번 `Font.custom("Pretendard-Bold", size: 24)`로 쓰면 오타·불일치가 생기기 쉽다. **Extension + static 프로퍼티**로 한 번 정의해두면 `.pretendardBold24`처럼 자동완성으로 안전하게 사용할 수 있다.

```swift
// enum으로 폰트 무게 정의 → static 프로퍼티로 빠르게 접근
extension Font {
    enum Pretend { case bold, semibold, medium, regular, ... }
    static func pretend(type:size:) -> Font
    static var pretendardBold24: Font { .pretend(type: .bold, size: 24) }
}

// 사용: 간결하게 호출
Text("제목").font(.pretendardBold24)
```

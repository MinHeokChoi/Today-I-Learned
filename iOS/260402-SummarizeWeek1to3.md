# 🗂️ MegaBox 프로젝트 Week1 ~ Week3에서 배운 SwiftUI 핵심 요소  

## 1. 앱 구조 & 생명주기

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

| 개념 | 사용 위치 |
|------|----------|
| `.toolbar { ToolbarItem(placement:) }` | HomeView, MovieDetailView |
| `.sharedBackgroundVisibility(.hidden)` | 글래스 효과 제거 |
| `.navigationBarBackButtonHidden(true)` | 커스텀 뒤로가기 버튼 |
| `.navigationBarTitleDisplayMode(.inline)` | 타이틀 작게 표시 |
| `.safeAreaBar(edge: .top)` | 헤더 세그먼트 고정 |

---

## 8. 의존성 주입 (DI) 패턴

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

| 컴포넌트 | 파일 | 재사용 방법 |
|---------|------|------------|
| `ProfileHeaderView` | Components/ | 이름, 포인트, 회원정보 버튼 |
| `QuickActionButton` | Components/ | imageName만 바꿔서 4개 버튼 생성 |
| `StatItem` | MyPageView 내부 | title/value만 바꿔서 3개 항목 |
| `infoRow(label:value:)` | MovieDetailView | 6개 영화 정보 행 |

---

## 11. Font 확장 (Extension)

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

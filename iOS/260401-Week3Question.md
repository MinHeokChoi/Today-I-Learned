## 학습 목표

1. **TabView** — 탭 기반 화면 구성과 `selection ↔ value` 바인딩
2. **Toolbar & NavigationBar** — 시스템 UI 영역 이해와 커스텀
3. **Image 렌더링** — `.renderingMode`, modifier 체이닝 순서
4. **@Environment & DIContainer** — 의존성 주입과 중앙 상태 관리
5. **MVVM 패턴 적용** — Model/ViewModel/View 역할 분리
6. **ScrollViewReader** — 프로그래밍 방식 스크롤 제어
7. **Liquid Glass** — iOS 26 새로운 UI 효과

---

## 1. TabView — selection과 value의 관계

### 핵심 개념
TabView의 `selection`과 각 Tab의 `value`는 SwiftUI가 내부적으로 연결한 **매칭 시스템**이다.

```swift
TabView(selection: $selectedTab) {        // "지금 몇 번 탭이야?"
    Tab("홈", systemImage: "house", value: 0) {      // "나는 0번"
        HomeView()
    }
    Tab("바로 예매", systemImage: "play.laptopcomputer", value: 1) {  // "나는 1번"
        ReservationView()
    }
}
```

### 양방향 동작
- **사용자가 탭 터치** → 해당 Tab의 `value`가 `selection`에 자동 대입
- **코드에서 `selectedTab = 1`** → `value: 1`인 Tab이 자동으로 화면에 표시

### 동일 패턴이 여러 곳에 적용됨

| 부모 컴포넌트 | 선택 상태 | 자식 식별자 |
|---|---|---|
| `TabView` | `selection` | `Tab(value:)` |
| `Picker` | `selection` | `.tag()` |
| `TabView(.page)` | `selection` | `.tag()` |

> 이 패턴은 SwiftUI 문서를 읽어야 아는 **프레임워크 약속(Convention)** 이다.

---

## 2. Toolbar vs NavigationBar

### 관계
- **NavigationBar** = 화면 상단의 **영역(공간)** 자체
- **`.toolbar { }`** = 그 영역에 **아이템을 넣는 방법**

```
NavigationStack
├── NavigationBar (시스템이 자동 생성하는 영역)
│   ├── topBarLeading   ← ToolbarItem 배치 가능
│   ├── principal        ← 가운데
│   └── topBarTrailing   ← 오른쪽
├── 콘텐츠
└── bottomBar
```

비유: NavigationBar는 **선반**, `.toolbar`는 선반 위에 **물건을 올리는 행위**

### 주요 modifier

```swift
// NavigationBar 배경을 투명하게 (영역은 그대로 차지함!)
.toolbarBackgroundVisibility(.hidden, for: .navigationBar)

// Toolbar 아이템의 글래스 효과 제거
.toolbar {
    ToolbarItem(placement: .topBarLeading) {
        Image("logo")
    }
    .sharedBackgroundVisibility(.hidden)
}
```

> `.toolbarBackgroundVisibility(.hidden)`은 **투명 망토**와 같다 — 안 보이지만 자리는 차지한다.

---

## 3. Image 렌더링과 Modifier 순서

### renderingMode

| 모드 | 동작 | 사용처 |
|---|---|---|
| `.template` | 이미지 색상 무시, **tint 색**으로 칠함 | SF Symbols, 단색 아이콘 |
| `.original` | 이미지 **원래 색상** 그대로 | 로고, 사진 등 고유 색상 이미지 |

- **Toolbar은 기본 `.template`** → 로고에는 `.renderingMode(.original)` 필수
- **의도적으로 `.template` 활용** → 특별관 로고 tint 처리에 유용

```swift
// Toolbar 로고 — 원본 색상 유지
Image("megaboxLogo")
    .renderingMode(.original)
    .resizable()
    .scaledToFit()

// 특별관 로고 — 선택 상태에 따라 tint
Image(theater.logo)
    .renderingMode(.template)
    .foregroundStyle(isSelected ? .primary : .gray)
```

### Modifier 체이닝 순서가 중요하다

```swift
// ❌ 에러 — .resizable() 이후 Image 전용 메서드 사용 불가
Image("logo")
    .resizable()              // Image → some View로 변환
    .renderingMode(.original) // ❌ some View에는 이 메서드 없음!

// ✅ 정상 — Image 전용 메서드를 먼저 호출
Image("logo")
    .renderingMode(.original) // Image 전용 → 아직 Image 타입
    .resizable()              // Image 전용 → some View로 변환
    .scaledToFit()            // some View 메서드
```

> **규칙: `.renderingMode()`, `.resizable()`은 Image 전용 메서드 → 체인 앞쪽에!**

---

## 4. @Environment와 DIContainer — 중앙 상태 관리

### @Environment 주입 원리

```swift
// 주입하는 쪽 (부모)
MainTabView()
    .environment(DIContainer())

// 받는 쪽 (자식)
@Environment(DIContainer.self) private var container
```

**실제 앱과 Preview 모두** 주입이 필요하다:

```swift
// 실제 앱: App에서 주입
@main struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            MainTabView()
                .environment(DIContainer())
        }
    }
}

// Preview: Preview 블록에서 주입
#Preview {
    MainTabView()
        .environment(DIContainer())  // 이게 없으면 크래시!
}
```

### DIContainer로 탭 간 데이터 전달

```swift
@Observable
class DIContainer {
    var selectedTab: Int = 0
    var selectedMovieForReservation: MovieModel? = nil
    let homeRouter = NavigationRouter<HomeRoute>()
    let myPageRouter = NavigationRouter<MyPageRoute>()

    func resetAll() {
        selectedTab = 0
        homeRouter.reset()
        myPageRouter.reset()
    }
}
```

**탭 간 이동 예시 (홈 → 바로 예매):**

```swift
// HomeView에서
Button("바로 예매") {
    container.selectedMovieForReservation = movie
    container.selectedTab = 1  // 예매 탭으로 전환
}
```

> TabView의 탭 전환은 **수평 이동** — "뒤로가기" 개념이 없다. NavigationStack의 push/pop은 **수직(스택) 이동**.

### 로그아웃 시 상태 초기화

```swift
// ViewModel에서 처리 (MVVM 준수)
func logout(container: DIContainer) {
    container.resetAll()    // 모든 라우터 초기화 + 탭 리셋
    isLoggedIn = false
}
```

---

## 5. MVVM 패턴 — 데이터는 어디에?

### 역할 분리 원칙

| 레이어 | 역할 | 예시 |
|---|---|---|
| **Model** | 데이터 구조 정의 | `MovieModel`, `TheaterModel` |
| **ViewModel** | 데이터 보관 + 비즈니스 로직 | `HomeViewModel` (영화 목록, 극장 목록) |
| **View** | 화면 표시 + 사용자 입력 전달 | `HomeView` (UI 렌더링만) |

### 실제 적용 — 특별관 데이터

```swift
// ❌ View에 데이터 직접 선언 — MVVM 위반
struct HomeView: View {
    private let theaters = [("Dolby", "card1", "name1")]  // View가 데이터 소유
}

// ✅ Model + ViewModel로 분리
// Model
struct TheaterModel {
    let logo: String
    let card: String
    let name: String
    let title: String
    let description: String
}

// ViewModel
@Observable
class HomeViewModel {
    let theaters: [TheaterModel] = [
        TheaterModel(logo: "Dolby Cinema 로고", card: "Dolby Cinema", ...)
    ]
}

// View — viewModel에서 가져다 쓰기만
ForEach(Array(viewModel.theaters.enumerated()), id: \.offset) { ... }
```

### 무비차트 / 상영예정 분리

```swift
// ViewModel
@Observable
class HomeViewModel {
    let movies: [MovieModel] = [...]           // 현재 상영작
    let upcomingMovies: [MovieModel] = [...]   // 상영 예정작

    func currentMovies(for segment: Int) -> [MovieModel] {
        segment == 0 ? movies : upcomingMovies
    }
}

// View — segment에 따라 다른 목록 표시
ForEach(viewModel.currentMovies(for: selectedSegment)) { movie in ... }
```

### View에서 로직 분리하기

```swift
// ❌ View가 직접 로직 실행
Button(action: {
    container.resetAll()
    isLoggedIn = false
})

// ✅ ViewModel이 처리, View는 호출만
Button(action: {
    viewModel.logout(container: container)  // "해줘" 한마디
})
```

> **View는 "뭘 할지" 결정, ViewModel은 "어떻게 할지" 실행**

---

## 6. ScrollViewReader — 프로그래밍 방식 스크롤 제어

### 개념
`ScrollViewReader`는 **proxy(대리인, 리모컨)** 를 제공해서 코드로 스크롤 위치를 제어할 수 있게 해준다.

```swift
ScrollViewReader { proxy in
    ScrollView(.horizontal) {
        LazyHStack {
            ForEach(0..<10) { index in
                Text("Item \(index)")
                    .id(index)       // 각 아이템에 id 부여
            }
        }
    }

    Button("5번으로 이동") {
        withAnimation {
            proxy.scrollTo(5, anchor: .center)
        }
    }
}
```

> **proxy = 대리인**. ScrollView를 직접 건드릴 수 없으니, proxy에게 "5번으로 가!" 명령을 내리는 것.

---

## 7. UI 스타일링 기법 모음

### ZStack으로 이미지 위 텍스트 오버레이

```swift
ZStack(alignment: .bottomLeading) {
    Image(theater.card)
        .resizable()
        .aspectRatio(contentMode: .fill)

    // 하단 그라데이션 (글씨 가독성용)
    LinearGradient(
        colors: [.clear, .black.opacity(0.7)],
        startPoint: .center,
        endPoint: .bottom
    )

    // 텍스트
    VStack(alignment: .leading, spacing: 4) {
        Text("DOLBY CINEMA")
            .font(.custom("Pretendard-Bold", size: 28))
        Text("완벽한 영화 관람을 완성하는\n하이엔드 시네마")
            .font(.custom("Pretendard-Medium", size: 18))
    }
    .foregroundStyle(.white)
    .padding(12)
}
```

### 원형 배경 + 그림자 + 테두리 (뉴모피즘 스타일)

```swift
Image(theater.logo)
    .resizable()
    .scaledToFit()
    .frame(width: 60, height: 60)
    .padding(10)
    .background {
        if isSelected {
            // 선택됨 — 볼록한 느낌
            Circle()
                .fill(Color.white)
                .shadow(color: .black.opacity(0.08), radius: 6, x: 4, y: 4)
                .shadow(color: .white.opacity(0.9), radius: 6, x: -4, y: -4)
        } else {
            // 선택 안 됨 — 오목한 느낌 (inner shadow)
            Circle()
                .fill(
                    Color(.gray02)
                        .shadow(.inner(color: .black.opacity(0.2), radius: 5, x: 4, y: 4))
                        .shadow(.inner(color: .white.opacity(0.9), radius: 5, x: -4, y: -4))
                )
        }
    }
    .overlay(
        Circle()
            .stroke(
                LinearGradient(
                    colors: isSelected
                        ? [.white.opacity(0.8), .clear, .black.opacity(0.05)]
                        : [.black.opacity(0.15), .clear, .white.opacity(0.8)],
                    startPoint: .topLeading,
                    endPoint: .bottomTrailing
                ),
                lineWidth: 1.5
            )
    )
```

### .shadow() 파라미터

```swift
.shadow(color: .black.opacity(0.1), radius: 3, x: 0, y: 1)
//      색상                         퍼짐정도  가로  세로(아래로)
```

---

## 8. Liquid Glass (iOS 26+)

### 기본 적용

```swift
// 캡슐 모양 글래스
Text("Hello").padding().glassEffect()

// 원형 글래스
Image("logo").padding().glassEffect(in: .circle)

// 커스텀 모양 + 틴트 + 터치 반응
Text("Hello").padding()
    .glassEffect(.regular.tint(.orange).interactive())

// 버튼에 글래스 스타일
Button("확인") { }.buttonStyle(.glass)
```

### Toolbar은 자동, View body는 수동

| 위치 | Liquid Glass 적용 |
|---|---|
| `.toolbar { }` 안 | **자동** (시스템 관리 영역) |
| View body 안 | **수동** (`.glassEffect()` 또는 `.buttonStyle(.glass)`) |

---

## 9. 같은 View 안에서 @State 접근 — 바인딩 불필요

```swift
struct HomeView: View {
    @State private var selectedTheaterIndex: Int = 0

    var body: some View {
        // ✅ 같은 View 안 — 직접 읽기/쓰기
        if selectedTheaterIndex == 0 { ... }           // 읽기
        .onTapGesture { selectedTheaterIndex = index } // 쓰기
        TabView(selection: $selectedTheaterIndex)       // $는 자식 뷰에게 넘길 때

        // $바인딩이 필요한 경우 — 다른 View에게 수정 권한 위임
        ChildView(index: $selectedTheaterIndex)
    }
}
```

> **$바인딩은 다른 View에게 수정 권한을 넘길 때만** 필요하다.

---

## 10. 기타 팁

### AppIcon 세트는 필수
- `Assets.xcassets`에 `AppIcon` 세트가 없으면 빌드 에러
- 이미지가 비어 있어도 **세트 자체는 존재해야** 함

### TabView(.page) 인디케이터
- `.page` 스타일은 **기본으로 페이지 닷 제공** — 직접 만들 필요 없음
- `.tabViewStyle(.page(indexDisplayMode: .never))` — 숨기기
- `.tabViewStyle(.page(indexDisplayMode: .always))` — 항상 표시

### TabView의 tag가 중요한 이유

```swift
TabView(selection: $selectedTheaterIndex) {
    ForEach(...) { index, theater in
        Image(theater.card).tag(index)  // selection과 tag가 양방향 연결
    }
}
```

- 로고 탭 → `selectedTheaterIndex` 변경 → 해당 `.tag()` 카드로 이동
- 카드 스와이프 → `.tag()` 값이 `selectedTheaterIndex`에 자동 대입

---

## 프로젝트 구조 (MVVM + Router)

```
UMC_MegaBox/Sources/
├── Models/
│   ├── MovieModel.swift        — 영화 데이터 구조
│   └── TheaterModel.swift      — 특별관 데이터 구조
├── ViewModels/
│   └── HomeViewModel.swift     — 영화/극장 목록, 세그먼트 필터링
├── Views/
│   ├── MainTabView.swift       — 4개 탭 구성, DIContainer 주입
│   ├── HomeView.swift          — 홈 화면 (무비차트, 특별관)
│   ├── MovieDetailView.swift   — 영화 상세 (상세정보/실관람평)
│   ├── ReservationView.swift
│   ├── MobileOrderView.swift
│   └── MyPageView.swift
├── Router/
│   ├── NavigationRouter.swift  — 제네릭 라우터 (push/pop/reset)
│   └── Routes.swift            — HomeRoute, MyPageRoute 정의
└── DIContainer.swift           — 탭 선택, 라우터 중앙 관리, resetAll()
```

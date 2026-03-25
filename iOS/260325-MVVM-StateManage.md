# SwiftUI MVVM 아키텍처와 상태 관리

## MVVM 패턴

MVVM(Model-View-ViewModel)은 UI와 데이터 로직을 분리하는 아키텍처 패턴이다.

**Model**은 앱의 데이터 구조를 정의하고, API 통신 등 데이터 관련 로직을 담당한다. UI와 직접 연결되지 않는다.

**ViewModel**은 Model과 View 사이의 중간 다리 역할을 한다. Model에서 받아온 데이터를 가공하여 View에 전달하며, 비즈니스 로직을 포함하되 UI와는 독립적이다. View가 어떻게 데이터를 표시할지는 관여하지 않는다.

**View**는 ViewModel에서 가공한 데이터를 UI로 표현하는 역할만 담당한다. 데이터 처리 로직을 포함하지 않는다.

### 데이터 흐름

```
View → (사용자 액션) → ViewModel → (데이터 요청) → Model
Model → (데이터 반환) → ViewModel → (가공된 데이터) → View → UI 업데이트
```

View는 ViewModel만 알고, ViewModel은 Model만 안다. View가 Model을 직접 참조하지 않는 것이 핵심이다.

### MVVM이 필요한 이유

기존 UIKit의 MVC 패턴에서는 ViewController가 데이터 처리, 비즈니스 로직, UI를 모두 담당하면서 비대해지는 문제(Massive ViewController)가 있었다. MVVM은 ViewModel로 로직을 분리하여 이를 해결하고, 테스트 용이성과 유지보수성을 높인다.

### 실제 프로젝트 폴더 구조 예시

```
MyApp/
├── Model/
│   ├── User.swift          // 데이터 구조 정의
│   └── APIService.swift    // 네트워크 통신
├── ViewModel/
│   ├── UserViewModel.swift // 비즈니스 로직, 상태 관리
│   └── HomeViewModel.swift
├── View/
│   ├── UserView.swift      // UI 구성
│   └── HomeView.swift
└── MyApp.swift
```

---

## SwiftUI 상태 관리

상태(State)란 UI에 영향을 주는 데이터다. SwiftUI는 선언형 프레임워크이므로, 상태가 바뀌면 자동으로 해당 뷰의 `body`를 다시 평가하여 UI를 갱신한다.

### 프로퍼티(Property)란?

클래스나 구조체 안에 선언된 변수/상수를 프로퍼티라고 부른다. 함수 안에서 선언하는 지역 변수와 구분되는 개념이다.

```swift
struct SomeView: View {
    var name = "홍길동"       // 프로퍼티 (구조체에 속한 변수)
    let age = 20             // 프로퍼티

    func doSomething() {
        var localVar = 0     // 지역 변수, 프로퍼티 아님
    }
}
```

이후 나오는 `@State`, `@Binding` 등은 이 프로퍼티 앞에 붙여서 특별한 동작을 부여하는 **속성 래퍼(Property Wrapper)**다.

---

### @State / @Binding

뷰 내부의 간단한 로컬 상태를 관리할 때 사용한다.

#### @State

```swift
struct CounterView: View {
    @State private var text: String = ""

    var body: some View {
        VStack {
            Text("텍스트 내용: \(text)")
            TextField("입력하세요", text: $text)
        }
    }
}
```

**@State 특징:**

- 뷰 내부에서 값을 소유하며, SwiftUI가 별도 저장소에서 생명주기를 관리한다
- 값이 변경되면 자동으로 UI가 업데이트된다
- 뷰 구조체가 재생성되어도 `@State` 값은 SwiftUI가 유지해준다
- 해당 뷰만 소유하는 내부 상태이므로 `private`으로 선언하는 것이 권장된다

#### @Binding과 $ 기호

`@Binding`은 실제 값을 저장하지 않고 상위 뷰의 `@State`를 참조한다. 원본 데이터에 대한 읽기/쓰기 접근을 제공한다.

핵심은 **`$`를 붙여서 넘겨야 하위 뷰에서 원본을 수정할 수 있다**는 것이다.

```swift
// $ 없이 넘기면 → 값의 복사본이 전달됨 → 원본 수정 불가
ChildView(name: name)

// $ 붙여서 넘기면 → 원본에 대한 참조가 전달됨 → 원본 수정 가능
ChildView(name: $name)
```

```swift
struct ParentView: View {
    @State private var isClicked: Bool = false
    var body: some View {
        VStack {
            Text("현재 값: \(String(isClicked))")
            CustomButton(isClicked: $isClicked) // $로 바인딩 전달
        }
    }
}

struct CustomButton: View {
    @Binding var isClicked: Bool  // 상위 뷰의 State를 참조
    var body: some View {
        Button("토글") { isClicked.toggle() } // 원본이 수정됨
    }
}
```

하위 뷰에서 `@Binding` 값을 변경하면 상위 뷰의 `@State`가 변경되고, 양쪽 뷰 모두 UI가 자동 갱신된다.

`TextField`, `Toggle`, `Stepper` 등 사용자 입력을 받는 컴포넌트는 값을 직접 변경해야 하므로 `Binding`을 요구한다.

`@State`/`@Binding`은 단순한 로컬 상태에 적합하지만, 복잡한 비즈니스 로직이 얽힌 상태 관리에는 한계가 있다. 이때 ViewModel로 상태 관리를 분리한다.

---

### @Observable 매크로 (iOS 17+, 권장 방식)

iOS 17부터 도입된 `@Observable` 매크로는 기존 `ObservableObject` + `@Published` 방식을 대폭 간소화한다.

```swift
// ViewModel
import Foundation

@Observable
class CounterViewModel {
    var count = 0           // 자동으로 변경 추적됨, @Published 불필요
    var title = "카운터"

    @ObservationIgnored     // 추적에서 제외하고 싶은 프로퍼티
    var internalCache = ""

    func increment() {
        count += 1          // 비즈니스 로직은 ViewModel 안에서 처리
    }
}
```

```swift
// View
struct CounterView: View {
    @State private var viewModel = CounterViewModel()

    var body: some View {
        VStack {
            Text("\(viewModel.count)")
            Button("증가") { viewModel.increment() }
        }
    }
}
```

**핵심 포인트:**

- `@Observable` 클래스를 뷰가 소유할 때는 **`@State`로 선언**해야 한다. 그래야 뷰가 리렌더링되어도 같은 인스턴스가 유지된다. `@State` 없이 일반 프로퍼티(`var viewModel = ...`)로 선언하면, 뷰 구조체가 재생성될 때 ViewModel도 새로 초기화될 수 있다.
- `@Observable`은 프로퍼티 단위로 변경을 추적한다. `body`에서 실제로 읽은 프로퍼티가 변경될 때만 뷰가 갱신되므로, 기존 `ObservableObject`보다 불필요한 리렌더링이 줄어든다.
- 추적에서 제외하려면 `@ObservationIgnored`를 사용한다.

**`@State`의 private 여부:**

- 단순 값 상태(`@State private var count = 0`)는 뷰 내부 전용이므로 `private` 권장
- ViewModel 소유 시에도 외부 주입이 필요 없다면 `private` 권장. 외부에서 ViewModel을 주입받아야 하는 경우에는 `private`을 생략한다

---

### @Binding vs @Bindable

둘 다 이름이 비슷하지만 대상과 용도가 다르다.

**@Binding**은 **하나의 값**(`String`, `Bool`, `Int` 등)에 대한 양방향 참조다. 상위 뷰에서 `$`를 붙여 바인딩을 넘겨주면, 하위 뷰에서 `@Binding`으로 받아 원본을 수정할 수 있다.

```swift
struct Parent: View {
    @State private var name = ""
    var body: some View {
        ChildView(name: $name)       // $를 붙여 값의 바인딩을 전달
    }
}

struct ChildView: View {
    @Binding var name: String        // 하나의 값을 참조
    var body: some View {
        TextField("이름", text: $name)
    }
}
```

**@Bindable**은 **`@Observable` 객체 전체**를 받아서, 그 안의 프로퍼티들에 `$` 바인딩 접근을 가능하게 해준다. iOS 17+ 전용이다.

```swift
struct Parent: View {
    @State private var vm = CounterViewModel()
    var body: some View {
        ChildView(vm: vm)            // 객체 자체를 전달 ($가 아님)
    }
}

struct ChildView: View {
    @Bindable var vm: CounterViewModel   // Observable 객체를 받음
    var body: some View {
        TextField("제목", text: $vm.title)   // $객체.프로퍼티로 바인딩 접근
        Stepper("값", value: $vm.count)
    }
}
```

정리하면, `@Binding`은 `$값`을 전달받아 그 값 하나를 수정하고, `@Bindable`은 객체를 전달받은 뒤 `$객체.프로퍼티` 형태로 내부 프로퍼티에 바인딩 접근한다. 자식 뷰에서 `@Observable` 객체의 프로퍼티를 읽기만 하거나 메서드만 호출한다면 `@Bindable` 없이 `let`으로 받아도 된다. `$` 바인딩이 필요한 순간에만 `@Bindable`을 쓴다.

---

### 부모-자식 뷰 간 ViewModel 공유

**자식이 수정 가능한 경우 — `@Bindable` 사용:**

```swift
struct ParentView: View {
    @State private var counter = CounterViewModel()
    var body: some View {
        CounterEditView(counter: counter)
    }
}

struct CounterEditView: View {
    @Bindable var counter: CounterViewModel
    var body: some View {
        Stepper("count: \(counter.count)", value: $counter.count)
        TextField("제목", text: $counter.title)
    }
}
```

**자식이 읽기만 하는 경우 — plain 선언:**

```swift
struct DetailView: View {
    let counter: CounterViewModel  // 읽기 전용
    var body: some View {
        Text("\(counter.count)")          // 값 변경 시 자동 갱신됨
        Button("증가") { counter.increment() } // 메서드 호출도 가능
    }
}
```

`let`으로 선언해도 `@Observable` 객체의 변경은 자동으로 감지된다. `@Observable` 매크로가 프로퍼티 접근 자체를 추적하기 때문에, 뷰의 `body`에서 해당 프로퍼티를 읽기만 해도 변경 시 자동으로 UI가 갱신된다. `$` 바인딩이 필요 없다면 이 방식으로 충분하다.

---

### Environment를 활용한 ViewModel 공유

여러 뷰가 하나의 ViewModel을 참조해야 하거나 뷰 계층이 깊을 때, 일일이 전달하는 대신 `.environment()`로 주입한다.

```swift
struct ParentView: View {
    @State private var counter = CounterViewModel()
    var body: some View {
        VStack {
            Text("부모: \(counter.count)")
            ChildView()
                .environment(counter)  // 환경에 주입
        }
    }
}

struct ChildView: View {
    @Environment(CounterViewModel.self) private var counter

    var body: some View {
        // 읽기만 할 때는 그대로 사용
        Text("자식: \(counter.count)")

        // $바인딩이 필요할 때는 body 안에서 @Bindable로 래핑
        @Bindable var bindableCounter = counter
        Stepper("값 변경", value: $bindableCounter.count)
    }
}
```

**주의:** `@Environment`로 가져온 객체에서 `$` 바인딩을 사용하려면 `body` 안에서 `@Bindable var`로 래핑해야 한다. 읽기나 메서드 호출만 한다면 래핑 없이 바로 사용 가능하다.

#### SwiftUI 기본 제공 환경값

SwiftUI는 시스템 정보를 Environment에 기본 제공한다.

```swift
@Environment(\.colorScheme) var colorScheme      // .dark 또는 .light
@Environment(\.dismiss) private var dismiss       // 현재 뷰 닫기
@Environment(\.horizontalSizeClass) var sizeClass // 화면 크기 클래스
```

커스텀 ViewModel을 환경에 넣을 때는 `.environment(객체)` 형태이고, 시스템 환경값은 `@Environment(\.keyPath)` 형태인 점이 다르다.

---

### @AppStorage

`UserDefaults`를 SwiftUI 방식으로 간편하게 사용하는 속성 래퍼다. 앱을 종료해도 값이 유지되며, 값 변경 시 자동으로 UI가 업데이트된다.

```swift
struct SettingsView: View {
    @AppStorage("isDarkMode") private var isDarkMode: Bool = false
    @AppStorage("username") private var username: String = "Guest"
    @AppStorage("fontSize") private var fontSize: Double = 14.0

    var body: some View {
        VStack {
            Toggle("다크 모드", isOn: $isDarkMode)
            TextField("이름", text: $username)
            Slider(value: $fontSize, in: 10...30)
            Text("미리보기").font(.system(size: fontSize))
        }
    }
}
```

**특징:**

- 키(문자열)를 기준으로 `UserDefaults`에 자동 저장/로딩된다
- 같은 키를 사용하는 `@AppStorage`는 서로 다른 뷰에 있어도 값이 동기화된다
- `String`, `Int`, `Double`, `Bool`, `URL`, `Data` 등 기본 타입을 지원한다
- 가벼운 사용자 설정(테마, 온보딩 여부 등)에 적합하며, 대량의 데이터 저장에는 부적합하다

---

## 정리

| 용도 | 권장 방식 |
|---|---|
| 뷰 내부 간단한 상태 | `@State` / `@Binding` |
| ViewModel 상태 관리 | `@Observable` 매크로 + `@State`로 소유 |
| 값 하나의 양방향 참조 (하위 뷰) | `@Binding` — 상위에서 `$`로 전달 |
| Observable 객체의 프로퍼티 바인딩 | `@Bindable` — 객체를 받아 `$객체.프로퍼티` 접근 |
| 하위 뷰에서 읽기/메서드 호출만 | `let` (plain) |
| 깊은 뷰 계층에 ViewModel 공유 | `.environment()` + `@Environment` |
| Environment에서 바인딩 필요 시 | `body` 안에서 `@Bindable var` 래핑 |
| 변경 추적 제외 프로퍼티 | `@ObservationIgnored` |
| 영구 저장 (가벼운 설정값) | `@AppStorage` |

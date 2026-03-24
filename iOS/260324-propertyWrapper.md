# [SwiftUI] 데이터 관리와 속성 래퍼(Property Wrapper) 총정리

## 1. 속성 래퍼(Property Wrapper)란?
* **정의:** 변수를 읽고 쓸 때 필요한 '반복적인 로직'을 캡슐화하여 재사용할 수 있게 해주는 기능.
* **비유:** 변수에 씌우는 '특별한 기능이 달린 포장지' (예: `@` 기호를 붙여 사용)
* **장점:** 복잡한 보일러플레이트 코드를 줄이고 코드를 깔끔하게 유지할 수 있다.

---

## 2. @AppStorage
사용자의 간단한 설정이나 상태를 기기에 영구적으로 저장할 때 사용하는 속성 래퍼.

* **동작 원리:** iOS 기본 저장소인 `UserDefaults`와 자동으로 연결된다.
* **장점:** 값을 변경하면 자동 저장되고, 해당 값을 사용하는 화면(View)도 자동 갱신된다.
* **주의사항:** * 암호화되지 않으므로 **비밀번호 등 민감한 정보 저장 절대 금지** (대신 `Keychain` 사용 권장).
  * 문자열(String), 숫자(Int, Double), 참/거짓(Bool) 같은 가벼운 데이터에만 적합하다.

**사용 예시:**
```swift
@AppStorage("username") var myName: String = "익명"

// 화면에서 이름 변경 시 기기 저장소에 자동 반영 및 화면 갱신
TextField("이름", text: $myName)
```

---

## 3. 앱 전역 데이터 공유 (Prop Drilling 해결)
상위 뷰에서 하위 뷰로 데이터를 일일이 전달하는 불편함(Prop Drilling)을 해결하기 위해, 앱의 '공간(Environment)'에 데이터를 띄워두고 필요한 곳에서만 꺼내 쓰는 방식.

### 💡 기존 방식 (iOS 16 이하)
두 가지를 엄격하게 구분해서 사용해야 했다.
* **`@Environment`:** 시스템이 제공하는 설정값 (다크모드, 언어 설정 등)을 읽어올 때.
* **`@EnvironmentObject`:** 개발자가 직접 만든 복잡한 커스텀 데이터(`ObservableObject`)를 공유할 때. (최상위 뷰에서 `.environmentObject()`로 반드시 주입해야 함)

### 🚀 최신 방식 (iOS 17 이상: 매크로 대통합)
`@Observable` 매크로가 도입되면서 문법이 아주 단순해지고 성능이 대폭 향상되었다.
* **데이터 모델링:** `@Observable` 하나로 끝 (`@Published` 불필요).
* **데이터 주입:** `.environment()` 하나로 통합.
* **데이터 읽기:** 커스텀 데이터든 시스템 설정이든 모두 `@Environment`로 통일.
* **성능 향상:** 뷰에서 실제로 사용하는 변수가 변경될 때만 화면을 새로고침하여 매우 효율적이다.

**사용 예시 (iOS 17+):**
```swift
import Observation

@Observable 
class UserSettings {
    var name: String = "김스위프트"
}

// 1. 최상위 뷰에서 주입
ContentView()
    .environment(UserSettings())

// 2. 하위 뷰에서 꺼내 쓰기
@Environment(UserSettings.self) private var settings
```

---

## 4. @Bindable (iOS 17+)
`@Environment`로 읽어온 데이터나 상위 뷰에서 전달받은 데이터를 화면 UI(예: `TextField`, `Toggle`)를 통해 **'직접 양방향 수정'**하고 싶을 때 사용하는 속성 래퍼.

* **필요성:** 내가 직접 만든 데이터(주인인 `@State`)가 아니라 남에게 빌려온 데이터는 함부로 바인딩(`$`) 기호를 붙여 수정할 수 없다. 이때 `@Bindable`로 감싸주면 수정 권한이 생긴다.
* **주의사항:** 데이터를 단순히 화면에 보여주기만 할 때는 필요 없다. 사용자의 입력으로 값이 변해야 할 때만 사용한다.

**사용 예시:**
```swift
@Environment(UserSettings.self) private var settings

var body: some View {
    // 수정을 위해 @Bindable로 한 번 감싸기
    @Bindable var bindableSettings = settings
    
    // $ 기호를 사용하여 양방향 바인딩
    TextField("이름 수정", text: $bindableSettings.name)
}
```

---
date: 2026-04-02
tags: #SwiftUI #AppStructure #AppLifecycle #Scene #View #AppStorage
---

# [SwiftUI] App 구조와 생명주기 완벽 이해: 극장 비유로 끝내기

## 🎯 핵심 요약
SwiftUI 앱은 `App`(극장 건물) ➡️ `Scene`(무대) ➡️ `View`(배우)의 논리적인 계층 구조로 이루어집니다. `@main` 팻말을 통해 앱이 시작되며, 앱 전반의 상태(`@AppStorage` 등) 변화를 감지해 `RootView`에서 조건부로 화면을 전환하는 선언형 UI의 강력함을 활용할 수 있다.

## 💻 핵심 코드 및 문법

```swift
import SwiftUI

// 1. 앱의 진입점 (극장 건물 & 사장님)
@main
struct UMCMegaBoxApp: App {
    var body: some Scene {
        // WindowGroup: 가장 기본이 되는 무대 장치 (기기에 따라 창 개수 조절)
        WindowGroup {
            // ⭐️ 화면 전환(라우팅) 로직을 전담하는 안내원 View를 호출
            RootView() 
        }
    }
}

// 2. 화면 전환 안내원 (단일 책임 & Preview 활용 가능)
struct RootView: View {
    // ⭐️ UserDefaults와 연결된 상태 변수 (앱을 껐다 켜도 유지됨)
    // 주의: 보안이 중요한 토큰/비밀번호는 반드시 Keychain을 사용할 것!
    @AppStorage("isLoggedIn") private var isLoggedIn: Bool = false
    
    var body: some View {
        // 상태(State)에 따라 SwiftUI가 알아서 화면을 렌더링함
        if isLoggedIn { 
            MainTabView() // 로그인 상태면 메인 화면으로
        } else { 
            LoginView()   // 로그아웃 상태면 로그인 화면으로
        }
    }
}
```

## 🤔 헷갈렸던 지점 & 깨달음

**1. WindowGroup과 다중 창(Window) 지원 여부**
* **Before:** `WindowGroup`에 View를 여러 개 넣거나 사용하면 아이폰에서도 창을 여러 개 띄울 수 있을 것이다.
* **After:** `WindowGroup`은 똑같은 무대를 복사할 '능력'은 있지만, 실제 허용 여부는 **운영체제(OS)**에 달려있다. 아이폰(iOS)은 규칙상 하나의 창만 허용하며, 아이패드나 맥(Mac) 환경에서는 기기가 허용하므로 다중 창 띄우기가 가능하다. (참고: 맥에서 문서 기반 앱을 만들 때는 `DocumentGroup` 무대를 사용한다.)

**2. @AppStorage와 자동 로그인의 보안 문제**
* **Before:** 자동 로그인을 구현할 때, 사용자의 로그인 정보나 보안 토큰을 `@AppStorage`에 쓱 저장해 두면 편리하겠다.
* **After:** `@AppStorage`(UserDefaults 기반)는 자물쇠가 없는 투명 상자와 같아서 해킹에 취약하다. 앱 실행 시 화면을 분기하기 위한 **단순 상태값(`isLoggedIn` = true/false)**만 여기에 저장하고, 진짜 중요한 **보안 토큰(열쇠)**은 철제 금고인 `Keychain`에 저장해야 한다.

**3. 화면 전환 로직을 RootView로 따로 분리하는 진짜 이유**
* **Before:** 어차피 앱 시작점에서 화면을 나누는 거면, `@main`이 있는 `App` 파일 안에 `if-else` 문을 바로 적어도 똑같지 않을까?
* **After:** `RootView`라는 View 껍데기로 분리해 내면 2가지 엄청난 장점이 생긴다. 
  1. 무거운 시뮬레이터 대신 **Xcode Preview(미리보기)**를 활용해 즉각적인 UI 테스트가 가능해진다.
  2. 사장님(`App`)과 티켓 안내원(`RootView`)의 역할을 나누는 **단일 책임 원칙(SRP)**을 지켜, 향후 온보딩 화면 등이 추가되어도 코드가 깔끔하게 유지보수된다.

## 🚀 다음 스텝
[x] * **프로젝트 적용:** 간단한 '다크모드 설정 토글' 버튼을 만들어 `@AppStorage("isDarkMode")`에 저장하고, 앱 전체 테마가 껐다 켜도 유지되는지 테스트해 보기.
[] * **심화 학습:** 실제 로그인 시스템과 유사하게, 단순 상태는 `@AppStorage`로 관리하되 임의의 가짜 토큰 문자열을 `Keychain`에 안전하게 쓰고 읽어오는 헬퍼(Helper) 클래스 만들어 보기.

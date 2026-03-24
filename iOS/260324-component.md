# 📱 SwiftUI 컴포넌트 설계 - ProfileHeaderView + QuickActionButton

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

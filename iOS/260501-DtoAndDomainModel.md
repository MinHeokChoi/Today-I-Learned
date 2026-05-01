# TIL - API 의존성을 줄이는 DTO와 Domain Model

## 오늘 배운 것

오늘은 API 응답 구조에 직접 의존하지 않는 iOS 앱 구조에 대해 학습했다.  
핵심은 서버 API 응답을 그대로 View나 ViewModel에서 사용하지 않고, `DTO`와 `Domain Model`을 분리하는 것이다.

API는 언제든지 바뀔 수 있다.  
필드명이 `user_name`에서 `name`으로 바뀌거나, 데이터 구조가 중첩 객체로 변경될 수도 있다.

이때 View와 ViewModel이 API 응답 구조에 직접 의존하고 있다면 수정 범위가 커진다.  
반대로 DTO와 Domain Model을 분리하면 API 변경의 영향을 DTO 또는 Mapper 계층으로 제한할 수 있다.

---

## Domain Model이란?

`Domain Model`은 앱 내부에서 사용하는 핵심 모델이다.  
서버 응답 구조가 아니라, 앱의 요구사항을 기준으로 설계한다.

예를 들어 앱에서 사용자에게 필요한 정보가 `id`, `name`, `profileImageURL`, `bio`라면 도메인 모델은 다음처럼 작성할 수 있다.

```swift
struct UserModel {
    let id: String
    let name: String
    let profileImageURL: String?
    let bio: String

    var displayName: String {
        name.isEmpty ? "익명 사용자" : name
    }

    var isProfileComplete: Bool {
        !name.isEmpty && !bio.isEmpty
    }
}
```

도메인 모델의 특징:

- 앱의 요구사항을 기준으로 설계한다.
- API 응답 필드명과 직접 연결되지 않는다.
- `displayName`, `isProfileComplete` 같은 비즈니스 로직을 가질 수 있다.
- View와 ViewModel이 사용하기 좋은 형태를 가진다.

---

## DTO란?

`DTO(Data Transfer Object)`는 API 응답 데이터를 받기 위한 객체다.  
서버에서 내려주는 JSON 구조와 필드명에 맞춰 작성한다.

예를 들어 API 응답이 다음과 같다고 하자.

```json
{
  "user_id": "1",
  "user_name": "Sophie",
  "profile_image": "https://example.com/sophie.jpg",
  "user_bio": "iOS Developer"
}
```

이 응답을 받기 위한 DTO는 다음처럼 작성할 수 있다.

```swift
struct UserDTO: Codable {
    let userId: String
    let name: String
    let profileImage: String?
    let userBio: String

    enum CodingKeys: String, CodingKey {
        case userId = "user_id"
        case name = "user_name"
        case profileImage = "profile_image"
        case userBio = "user_bio"
    }
}
```

DTO의 특징:

- API 응답 구조에 맞춘다.
- JSON 디코딩을 위해 `Codable` 또는 `Decodable`을 채택한다.
- 서버 필드명이 바뀌면 DTO가 영향을 받는다.
- 앱의 핵심 비즈니스 로직은 넣지 않는 것이 좋다.

---

## Codable, Decodable, Encodable

`Codable`은 Swift 타입을 외부 데이터와 변환할 수 있게 해주는 프로토콜이다.

```swift
typealias Codable = Decodable & Encodable
```

각 역할은 다음과 같다.

- `Decodable`: JSON 같은 외부 데이터를 Swift 타입으로 변환할 수 있음
- `Encodable`: Swift 타입을 JSON 같은 외부 데이터로 변환할 수 있음
- `Codable`: `Decodable`과 `Encodable`을 모두 포함

API 응답을 읽기만 한다면 `Decodable`만 사용해도 된다.

```swift
struct UserDTO: Decodable {
    let userId: String
}
```

하지만 보통은 나중에 서버로 데이터를 보낼 가능성도 고려해서 `Codable`을 자주 사용한다.

---

## CodingKeys란?

`CodingKeys`는 JSON의 key와 Swift 프로퍼티 이름을 연결하는 번역표 역할을 한다.

```swift
enum CodingKeys: String, CodingKey {
    case userId = "user_id"
    case name = "user_name"
}
```

위 코드는 다음 의미를 가진다.

```text
JSON의 "user_id"   -> Swift의 userId
JSON의 "user_name" -> Swift의 name
```

`CodingKey`는 인코딩과 디코딩에 사용할 key를 표현하기 위한 Swift 표준 프로토콜이다.

중요한 점은 `CodingKeys` 혼자 번역을 수행하는 것이 아니라는 점이다.  
실제로 JSON을 해석하는 것은 `JSONDecoder`이고, `CodingKeys`는 그 과정에서 사용되는 key 매핑표다.

---

## JSONDecoder().decode(_:from:)

다음 코드는 JSON 데이터를 `UserDTO` 타입으로 변환한다.

```swift
let userDTO = try JSONDecoder().decode(UserDTO.self, from: data)
```

분해하면 다음과 같다.

```swift
JSONDecoder()
```

JSON 데이터를 Swift 타입으로 바꿔주는 디코더를 생성한다.

```swift
.decode(UserDTO.self, from: data)
```

`data` 안에 들어 있는 JSON을 `UserDTO` 타입으로 해석한다.

```swift
UserDTO.self
```

`UserDTO` 타입 그 자체를 의미한다.  
즉, 디코더에게 "이 JSON을 UserDTO 타입으로 변환해줘"라고 알려주는 부분이다.

```swift
try
```

디코딩은 실패할 수 있기 때문에 `try`가 필요하다.  
예를 들어 JSON 필드명이 다르거나, 타입이 맞지 않거나, 필수 값이 없으면 실패할 수 있다.

보통은 다음처럼 `do-catch`와 함께 사용한다.

```swift
do {
    let userDTO = try JSONDecoder().decode(UserDTO.self, from: data)
    let user = userDTO.toDomain()
} catch {
    print("Decoding error:", error)
}
```

---

## DTO를 Domain Model로 변환하기

DTO는 서버 응답을 받기 위한 모델이고, Domain Model은 앱 내부에서 사용할 모델이다.  
따라서 DTO를 Domain Model로 변환하는 과정이 필요하다.

```swift
extension UserDTO {
    func toDomain() -> UserModel {
        UserModel(
            id: userId,
            name: name,
            profileImageURL: profileImage,
            bio: userBio
        )
    }
}
```

여기서 `-> UserModel`은 함수의 반환 타입을 의미한다.

```swift
func toDomain() -> UserModel
```

즉, `toDomain()` 함수는 실행 결과로 `UserModel` 타입의 값을 반환한다.

변환 관계는 다음과 같다.

```text
UserDTO.userId       -> UserModel.id
UserDTO.name         -> UserModel.name
UserDTO.profileImage -> UserModel.profileImageURL
UserDTO.userBio      -> UserModel.bio
```

이 함수는 이미 만들어진 DTO 값을 앱 내부에서 쓰기 좋은 Domain Model로 바꾸는 일반 Swift 함수다.  
따라서 `toDomain()` 자체에는 `Codable`이 필요하지 않다.

---

## extension을 사용하는 이유

`extension`은 기존 타입에 새로운 기능을 추가하는 문법이다.

```swift
extension UserDTO {
    func toDomain() -> UserModel {
        ...
    }
}
```

사실 `toDomain()`은 `UserDTO` 구조체 안에 바로 작성해도 된다.

```swift
struct UserDTO: Codable {
    let userId: String
    let name: String

    func toDomain() -> UserModel {
        UserModel(
            id: userId,
            name: name,
            profileImageURL: nil,
            bio: ""
        )
    }
}
```

하지만 `extension`으로 분리하면 역할을 나누어 읽기 좋다.

```text
struct UserDTO 본문:
API 응답 데이터 구조와 JSON 매핑 규칙

extension UserDTO:
DTO를 Domain Model로 변환하는 기능
```

---

## 직접 매핑과 Mapper 방식

DTO를 Domain Model로 변환하는 방법은 크게 두 가지가 있다.

### 1. 직접 매핑

DTO의 extension에 변환 함수를 작성하는 방식이다.

```swift
extension UserDTO {
    func toDomain() -> UserModel {
        UserModel(
            id: userId,
            name: name,
            profileImageURL: profileImage,
            bio: userBio
        )
    }
}
```

사용 예시:

```swift
let user = userDTO.toDomain()
```

장점:

- 간단하고 직관적이다.
- 코드가 짧다.
- DTO와 Domain Model이 거의 1:1로 대응될 때 좋다.

단점:

- DTO가 Domain Model을 알게 된다.
- 변환 로직이 복잡해지면 DTO extension이 무거워질 수 있다.

---

### 2. Mapper 방식

변환 책임을 별도의 타입으로 분리하는 방식이다.

```swift
struct UserMapper {
    static func toDomain(from dto: UserDTO) -> UserModel {
        UserModel(
            id: dto.userId,
            name: dto.name,
            profileImageURL: dto.profileImage,
            bio: dto.userBio
        )
    }
}
```

사용 예시:

```swift
let user = UserMapper.toDomain(from: userDTO)
```

장점:

- DTO는 API 응답 파싱에 집중할 수 있다.
- Domain Model은 앱의 핵심 로직에 집중할 수 있다.
- Mapper는 변환 책임만 가진다.
- 변환 로직이 복잡할 때 관리하기 좋다.

단점:

- 코드와 파일이 늘어난다.
- 단순한 변환에는 다소 과하게 느껴질 수 있다.

---

## API 필드명이 변경되었을 때

예를 들어 API 필드명이 `user_name`에서 `name`으로 바뀌었다고 하자.

변경 전:

```json
{
  "user_name": "Sophie"
}
```

변경 후:

```json
{
  "name": "Sophie"
}
```

계층화된 구조에서는 영향 범위가 제한된다.

- View: 변경 없음
- ViewModel: 변경 없음
- Domain Model: 변경 없음
- DTO: 수정 필요
- Mapper 또는 toDomain(): 필요한 경우 수정

즉, View는 여전히 `user.name`을 사용하고, ViewModel도 여전히 `UserModel`을 다룬다.  
API 필드명 변경은 DTO 계층에서 처리하면 된다.

---

## API에 직접 의존하는 구조의 문제

API 응답 필드명을 View나 ViewModel에서 직접 사용하면 서버 변경에 매우 취약해진다.

예를 들어 View에서 다음처럼 사용한다고 하자.

```swift
Text(viewModel.userName)
```

또는 서버 응답 구조에 가까운 값을 그대로 사용한다고 하자.

```swift
Text(viewModel.profile.user_name)
```

이 경우 API 필드명이 바뀌면 View 코드까지 수정해야 한다.  
즉, 서버 변경이 앱 전체로 퍼진다.

---

## MVVM + DTO 계층의 흐름

계층화된 구조에서는 데이터 흐름이 다음과 같다.

```text
API JSON
-> DTO
-> Mapper 또는 toDomain()
-> Domain Model
-> ViewModel
-> View
```

각 계층의 책임은 다음과 같다.

```text
DTO:
API 응답 구조를 담당한다.

Mapper 또는 toDomain():
DTO를 Domain Model로 변환한다.

Domain Model:
앱의 핵심 데이터와 비즈니스 로직을 담당한다.

ViewModel:
View에 필요한 상태와 동작을 관리한다.

View:
화면을 그린다.
```

이 구조의 핵심은 View와 ViewModel이 API 응답 구조를 직접 알지 않아도 된다는 것이다.

---

## 오늘의 핵심 정리

```text
DTO는 서버의 언어다.
Domain Model은 앱의 언어다.
Mapper 또는 toDomain()은 서버의 언어를 앱의 언어로 번역하는 역할이다.
```

API는 외부 세계이기 때문에 언제든지 바뀔 수 있다.  
따라서 앱의 핵심 로직과 UI가 API 응답 구조에 직접 의존하지 않도록 설계해야 한다.

DTO와 Domain Model을 분리하면 API 변경이 발생해도 View와 ViewModel의 수정 범위를 줄일 수 있다.  
이것이 MVVM과 DTO 계층을 함께 사용하는 중요한 이유다.

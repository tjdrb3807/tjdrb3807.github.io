---
layout: post
title: "FeelTalk - Clean Architecture 리팩토링"
sitemap: false
categories: [study, ios]
tags: [ios]
---

* toc
{:toc}

## ✍️ Introduction
이 글은 FeelTalk 프로젝트에 적용했던 Clean Architecture를 다시 검토하며,
의존성 방향이 잘못 설정된 지점을 바로잡은 리팩토링 과정을 정리한 기록입니다.

> 이 글에서는 Auth(Login, SignUp) 도메인을 중심으로 리팩토링을 진행했으며, 정리된 기준은 추후 다른 도메인에도 확장 적용할 예정입니다.

---
## 🧱 Background
프로젝트 초반 제가 이해한 Clean Architecture는 다음과 같습니다.

* **Presentation / Domain / Data 계층 분리**
* **Domain이 외부 데이터 소스에 직접 의존하지 않는 구조.**
<!-- * Domain이 저수준의 모듈에 직접 의존하지 않는 구조 -->

FeelTalk 프로젝트는 3개의 계층으로 분리되어 있고, Repository 인터페이스도 정의되어 있었기 때문에 Clean Architecture가 적용된 프로젝트라 생각했습니다.

하지만 Clean Architecture에 대한 이해를 넓히고 기존 코드를 다시 리뷰하는 과정에서, **Domain이 여전히 외부 프레임워크에 직접 의존**하고 있다는 사실을 발견했습니다.

이는 Clean Architecture의 핵심인 **Dependency Rule(의존성은 항상 안쪽 계층을 향해야 한다)**을 준수하지 못한 구조였습니다.

이 문제를 해결하기 위해 Auth Domain을 대상으로 구조 개선 리팩토링을 진행하게 되었습니다.


<!-- 처음에는 레이어를 분리하고 Repository 프로토콜을 정의했기 때문에
Clean Architecture가 적용되었다고 생각했습니다.

하지만 이후 코드를 다시 리뷰하면서,
레이어 분리와 의존성 방향은 별개의 문제라는 것을 깨달았습니다.

구조는 분리되어 있었지만,
Domain은 여전히 저장소, SDK, DTO와 같은 기술적 세부사항에 직접 의존하고 있었습니다. -->
---
## 1️⃣ 문제 상황
리팩토링 이전 Auth 도메인에는 공통적인 구조적 문제가 있었습니다.

**UseCase와 Entity가 저장소(Keychain), 외부 SDK(Firebase), API 스펙(DTO) 같은 구체적인 기술 구현에 직접 결합되어 있었습니다.**

즉, 계층은 분리되어 있었지만 **의존성 방향은 여전히 바깥(세부 구현)**을 향하고 있었습니다.

아래는 대표적인 사례들입니다.

### 1. KeychainRepository에 직접 결합
![Thumbnail](/assets/img/blog/ios/feelTalk_clean_architecture_refactoring_image01.png)

`KeychainRepository`는 JWT 토큰을 Keychain에 저장하는 저장소 클래스입니다.
기존 구조에서는 UseCase가 이 저장소 구현에 직접 결합되어 있었습니다.

저장 방식이 변경될 경우(예: 다른 Keychain 래퍼 또는 UserDefaults로 교체),
Domain 계층이 함께 수정되어야 하는 구조였습니다.

이는 고수준 계층인 Domain이 저수준 구현(Data)에 직접 의존하는 형태로,
Clean Architecture의 **Dependency Inversion Principle(DIP)**을 위반하는 설계입니다.

### 2. Firebase SDK 직접 사용
![Thumbnail](/assets/img/blog/ios/feelTalk_clean_architecture_refactoring_image02.png) 
기존 구조에서는 UseCase가 Firebase SDK의 구체적인 사용 방식에 직접 결합되어 있었습니다.

이로 인해 Push 구현을 교체하거나 제거할 경우 Domain 수정이 불가피했고, Domain 계층이 인프라 코드에 종속된 상태였습니다.

### 3. Domain에서 Request DTO를 변환 및 생성
![Thumbnail](/assets/img/blog/ios/feelTalk_clean_architecture_refactoring_image03.png)
![Thumbnail](/assets/img/blog/ios/feelTalk_clean_architecture_refactoring_image04.png) 
DTO는 API 스펙을 반영하는 Data Layer 모델입니다.
API는 언제든 변경될 수 있는 외부 계약(contract)에 해당합니다.

Domain에서 DTO를 직접 생성하거나 변환할 경우,
API 스펙 변경이 Domain 수정으로 이어질 수 있습니다.

이는 정책 계층이 외부 계약에 종속되는 구조입니다.
  
---
## 2️⃣ Refactoring Strategy
이번 리팩토링의 목표는 Domain이 저장소 구현, 외부 SDK, DTO와 같은 기술적 세부사항에 직접 의존하지 않도록 의존성 방향을 재정렬하는 것이었습니다.

이를 위해 다음과 같은 기준을 적용했습니다.

- Domain 계층에는 필요한 기능을 Protocol로 정의하여 추상화하고,
- 실제 저장 방식이나 외부 SDK 연동과 같은 구현은 Data 계층에서 담당하도록 분리했습니다.
- 또한 DTO 생성 및 변환 책임은 Repository로 이동시켜, API 스펙 변경이 Domain 계층으로 전파되지 않도록 설계했습니다.

---
## 3️⃣ Refactoring 
이번 이팩토링의 핵심은 
Domain이 기술적 세부사항에 직접 의존하지 않도록
의존성의 경계를 재정의하는 것이었습니다.

기존 구조에서 Domain이 의존하고 있던 요소는 크게 두 종류였습니다.
1. 인프라 의존성(저장소, 외부 SDK)
2. 외부 계약(API 스펙)

이에 따라 리팩토링도 두 단계로 나누어 진행했습니다.

### 1. 인프라 의존성 분리
#### 문제
UseCase는 다음과 같은 인프라 코드에 직접 의존하고 있었습니다.
* KeychainRepository(토큰 저장)
* FirebaseMessaging(FCM 토큰 조회 및 삭제)

이들은 모두 기술 선택에 해당하는 요소이며, 
비즈니스 정책과는 직접적인 관련이 없습니다.

하지만 기존 구조에서는 Domain이 이러한 인프라 구현을 직접 알고 있었고, 
* 저장 방식 변경
* Push 서비스 교체

와 같은 변화가 발생할 경우 Domain 코드 수정이 불가피한 구조였습니다.

이는 고수준 모듈(Domain)이 저수준 구현(Data)에 의존하는 형태로, 
**Dependency Inversion Principle(DIP)**을 위반하는 설계였습니다.

#### 해결
Domain 계층에 인프라 경계를 정의하는 인터페이스를 도입했습니다.

* 1. 토큰 저장 추상화(AuthTokenStore 도입)
기존 구조에서는 UseCase가 KeychainRepository에 직접 의존하고 있었습니다.

KeychainRepository는 토큰을 Keychain에 저장하는 Data 계층의 저장소 구현 클래스입니다.
하지만 Domain의 관심사는 "토큰을 저장한다"는 정책이지,
어떤 저장 기술을 사용하는지가 아닙니다.

이를 해결하기 위해 Domain 계층에 다음과 같은 인터페이스를 정의했습니다.

```swift
protocol AuthTokenStore {
    @discardableResult
    func saveAccessToken(_ token: String) -> Bool
    ...
}
```

해당 인터페이스는 저장 전략을 추상화하기 위한 경계(boundary)입니다.

이로 인해 UseCase는 더이상 Keychain이라는 구체 기술에 의존하지 않고, 
AuthTokenStore라는 추상에만 의존하게 되며, 

Data 계층에서 실제 저장 구현을 담당합니다.

```swift
final class KeychainAuthTokenStore: AuthTokenStore {
    @discardableResult
    func saveAccessToken(_ token: String) -> Bool {
        KeychainRepository.addItem(value: "accessToken", key: token)
    }
}
```

의존성 구조 변화

```
Before:
UseCase → KeychainRepository

After:
UseCase → AuthTokenStore (Protocol)
                 ↑
        KeychainAuthTokenStore (구현)
```

핵심은 고수준 모듈(UseCase)이 더 이상 저수준 구현(Keychain)에 의존하지 않는 구조로 변경되었다는 점입니다.

이로써 저장 전략이 변경되더라도
새로운 구현체를 추가하는 방식으로 확장이 가능해졌고, 
Domain 수정은 필요하지 않게 되었습니다.

* 2. Push 인프라 추상화(PushTokenProvier 도입)
기존 구조에서는 UseCase가 FirebaseMessaging을 직접 `import`하고,
FCM 토큰 조회 및 삭제를 수행하고 있었습니다.

Push 연동은 인프라 영역에 해당하며, Domain이 특정 SDK에 직접 결합될 이유는 없습니다.

이를 해결하기 위해 다음 인터페이스를 Domain 계층에 정의했습니다.

```swift
protocol PushTokenProvider {
    func fetchToken() -> Single<String?>
    func deleteToken() -> Single<Void>
}
```

해당 인터페이스를 통해 UseCase는 Firebase SDK를 알지 않으며, 
PushTokenProvier에만 의존합니다.

Data계층에서 Firebase 기반 구현체를 제공합니다.

```swift
final class FirebasePushTokenProvider: PushTokenProvider {
    ...
}
```

의존성 구조 변화

```
Before:
UseCase → FirebaseMessaging

After:
UseCase → PushTokenProvider (Protocol)
                ↑
   FirebasePushTokenProvider (구현)
```

이로써 Push 구현이 변경되더라도 Domain은 수정되지 않는 구조가 되었습니다.

### 2. 외부 계약(API 스펙) 의존성 분리
#### 문제
기존 구조에서는 Domain에서 RequestDTO를 직접 생성하거나,
Entity 내부에서 DTO 변환 로직을 보유하고 있었습니다.

DTO는 서버 API 스펙을 반영하는 Data 계층 모델이며,
외부 계약(contract)에 해당합니다.

API는 변경 가능성이 높은 영역입니다.
Domain이 DTO에 의존하면, API 변경이 정책 수정으로 이어지는 구조가 됩니다.

이는 정책 계층이 외부 계약에 종속된 상태입니다.

#### 해결
DTO 생성 및 변환 정책을 Repository(Data 계층)로 이동했습니다.

```swift
let requestDTO = SignUpRequestDTO(...)
repository.signUp(requestDTO)
```

의존성 구조 변화

```
Before:
Domain → DTO

After:
Domain → Repository (Protocol)
              ↓
        DTO 변환은 Data 계층에서 처리
```

이로써 API 스펙 변경은 Data 계층에만 영향을 미치게 되었고,
Domain은 정책에만 집중하느느 구조로 정리되었습니다.



<!-- ### 1. 저장소 의존성 제거(AuthTokenStore 도입)
기존 구조에서는 UseCase가 `KeychainRepository`에 직접 의존하고 있었습니다.

`KeychainRepository`는 토큰을 Keychain에 저장하는 Data 계층의 저장소 구현 클래스입니다.
하지만 Domain 계층의 관심사는 "토큰을 저장한다"는 정책이지, 어떤 저장 기술을 사용하는지가 아닙니다.

즉, 저장 방식은 세부 구현에 해당하며, Domain이 알 필요가 없는 영역입니다.

이를 해결하기 위해 Domain 계층에 다음과 같은 추상 인터페이스를 정의했습니다.

```swift
protocol AuthTokenStore {
    @discardableResult
    func saveAccessToken(_ token: String) -> Bool
    ...
}
```

해당 인터페이스는 저장 전략을 추상화하기 위한 경계(boundary)입니다.
UseCase는 더 이상 Keychain이라는 구체 기술에 의존하지 않고, `AuthTokenStore`라는 추상에만 의존하게 되었습니다.

Data 계층에서는 해당 인터페이스를 구현하도록 분리했습니다.

```swift
final class KeychainAuthToeknStore: AuthTokenStore {
    @discardableResult
    func saveAccessToken(_ token: String) -> Bool {
        KeychainRepository.addItem(value: "accessToken", key: token)
    }
    ...
}
```

이 작업으로 의존성 방향은 다음과 같이 재정렬되었습니다.

```
Before:
UseCase → KeychainRepository

After:
UseCase → AuthTokenStore (추상)
                 ↑
        KeychainAuthTokenStore (구현)
```

**핵심은 고주순 모듈(UseCase)이 더이상 저수준 구현(Keychain)에 의존하지 않는 구조로 바뀌었다는 점입니다.**

이로써 저장 전략이 변경되더라도 새로운 구현체를 추가하는 방식으로 확장할 수 있으며, Domain 계층은 수정되지 않습니다.

즉, 저장 기술과 비즈니스 정책이 분리되었고 의존성은 항상 안쪽 계층을 향하도록 정렬되었습니다.

### 2. Push 인프라 의존성 분리(PushTokenProvier 도입)



<!-- ### Token 저장소를 추상화 한다 (Keychain 분리)
기존에는 UseCase에서 KeychainRepository를 직접 호출하고 있었습니다.
KeychainRepository는 Data Layer의 구현체 이므로 Domain에서 알 필요가 없었습니다.

따라서 Domain Layer에 아래 인터페이스를 정의했습니다.

```swift
protocol AuthTokenStore {
    func saveAccessToken(_ token: String) -> Bool
    func loadAccessToken() -> String?
    func saveRefreshToken(_ token: String) -> Bool
    func loadRefreshToken() -> Sring?
    func saveExpiredTime(_ time: String) -> Bool
    func clearToken() -> Bool
}
```

그리고 Data Layer에서 Keychain 기반 구현체를 추가했습니다

```swift
final class KeychainAuthTokenStore: AuthTokenStore {
    // 내부에서만 KeychainRepository 사용 
}
```

### FCM Token 처리를 추상화(Firebase 분리)
기존에는 UseCase가 FirebaseMessaging를 직접 import하고, fcmToken 조회 / 삭제까지 처리했습니다.
이는 Domain이 외부 SDK에 종속되는 구조였습니다.

Domain에 아래 인터페이스를 정의했습니다.
```swift
protocol PushTokenProvier {
    func fetchToken() -> Single<String?>
    func deleteToken() -> Single<Void>
}
```

Data Layer에는 Firebase 구현체를 추가했습니다.
```swift
final class FirebasePushTokenProvider: PushTokenProvider {
    func fetchToken() -> Single<String?> { ... }
    func deleteToken() -> Single<Void> { ... }
}
```

결과적으로 Domain에서 import FirebaseMessaging를 제거할 수 있었습니다.

### UseCase에서 구체 구현을 제거하고, 의존성을 주입(DIP 적용)
이제 UseCase는 Keychain / Firebase 대신 인터페이스(프로토콜)에만 의존하도록 변경했습니다.

```swift
final class DefaultLoginUseCase: LoginUseCase {
    private let tokenStore: AuthTokenStore
    private let pushTokenProvider: PushTokenProvider

    init(
        loginRepository: L
    )
}
``` --> -->

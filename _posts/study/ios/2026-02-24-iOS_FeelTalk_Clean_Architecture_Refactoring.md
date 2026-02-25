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
이 글은 FeelTalk 프로젝트에 도입했던 Clean Architecture를 다시 점검하며, **의존성 방향이 잘못 설정된 부분을 바로잡은** 리팩토링 과정을 정리한 글입니다.

> 이 글에서는 Auth(Login, SignUp) 도메인을 중심으로 리팩토링을 진행했으며, 정리된 기준은 추후 다른 도메인에도 확장 적용할 예정입니다.

---
## 🧱 Background
프로젝트 초반 제가 이해한 Clean Architecture는 다음과 같습니다.

* **Presentation / Domain / Data 레이어를 구분**
* **Domain이 외부 데이터 소스에 직접 의존하지 않는 구조.**

FeelTalk 프로젝트는 3개의 레이어로 분리되어 있고, Repository 인터페이스도 정의되어 있었기 때문에 Clean Architecture가 적용된 프로젝트라 생각했습니다.

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

즉, 레이어는 분리되어 있었지만 **의존성 방향은 여전히 바깥(세부 구현)**을 향하고 있었습니다.

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

Domain에서 DTO직접 생성하거나 변환할 경우,
API 스펙 변경이 Domain 수정으로 이어질 수 있습니다.

이는 정책 계층이 외부 계약에 종속되는 구조입니다.
  
---
## 2️⃣ Refactoring Strategy
이번 리팩토링의 목표는 **Domain이 implementation detail(Repository, SDK, DTO)에 직접 의존하지 않도록 의존성 방향을 재정렬하는 것**이었습니다.

- Domain에는 필요한 기능을 Protocol로 정의하여 인터페이스화 하고
- Data는 이를 구현(concrete implementation)하도록 이동시켰습니다.
- DTO 변환/생성 책임은 Repository로 이동해 API 스펙 변경이 Domain에 전파되지 않도록 설계했습니다. 

### Token 저장소를 추상화 한다 (Keychain 분리)
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
```

---
layout: post
title: "FeelTalk - Clean Architecture 리팩토링"
sitemap: false
categories: [study, ios]
tags: [ios, clean-architecture, architecture, refactoring]
---

* toc
{:toc}

![Thumbnail](/assets/img/blog/ios/feelTalk_clean_architecture_refactoring_thumbnail.png)

## ✍️ Introduction
이 글은 FeelTalk 프로젝트에 적용했던 Clean Architecture 구조를 다시 검토하며,**잘못 설정된 의존성 방향을 안쪽(Domain)으로 바로잡은 리팩토링 과정**을 기록한 회고입니다.

> 이번 리팩토링은 Auth(Login, SignUp) Domain을 중심으로 진행했으며, 여기서 정립된 의존성 분리 기준은 추후 다른 Domain 영역에도 확장 적용할 예정입니다.

---

## 🧱 Background
프로젝트 초반 제가 이해한 Clean Architecture는 다음과 같습니다.
* **'Presentation / Domain / Data 계층의 분리'**
* **'Domain이 외부 기술에 의존하지 않는 것'**

FeelTalk 프로젝트는 계층별로 폴더를 나누고 Repository 인터페이스를 두었기에 원칙을 잘 지키고 있다고 생각했습니다.

하지만 Clean Architecture에 대한 견문을 넓히고, 코드를 다시 리뷰했을 때 **고수준 정책을 담당해야 할 Domain 계층이 여전히 외부 프레임워크와 세부 구현체에 직접 의존**하고 있었던 것을 발견했습니다.

계층은 나뉘었지만 의존성의 화살표는 여전히 바깥(세부 구현)을 향하는 상태였고, 이는 Clean Architecture의 핵심인 **의존성 규칙(Dependency Rule)**을 위반하는 설계였습니다.

---

## 1️⃣ Problem
리팩토링 이전, Auth Domain의 UseCase와 Entity는 아래와 같이 인프라 및 외부 계약에 강하게 결합되어 있었습니다.

1. **Keychain 구현체에 직접 결합** 
    ![Thumbnail](/assets/img/blog/ios/feelTalk_clean_architecture_refactoring_image01.png)
    UseCase 내부에서 토큰 저장소인 `KeychainRepository` 구체 클래스를 직접 호출했습니다. 저장 방식이 변경되면 Domain도 함께 수정되어야 하는 구조였습니다.
2. **Firebase SDK 침투**
   ![Thumbnail](/assets/img/blog/ios/feelTalk_clean_architecture_refactoring_image02.png)
   푸시 토큰 처리를 위해 UseCase 상단에 `import FirebaseMessaging`이 선언되어 있었습니다. Domain이 특정 외부 SDK에 종속된 상태였습니다.
1. **DTO 생성 및 변환 로직 포함** 
   ![Thumbnail](/assets/img/blog/ios/feelTalk_clean_architecture_refactoring_image03.png)

   ![Thumbnail](/assets/img/blog/ios/feelTalk_clean_architecture_refactoring_image04.png)
   API 스펙을 반영하는 Data 모델인 Request DTO를 Domain(Entity, UseCase)에서 직접 생성하고 변환했습니다. API 스펙이 변하면 비즈니스 로직(Domain)까지 흔들리게 됩니다.<br>
   결과적으로, 고수준(Domain)이 저수준(Data/Infra)에 의존하는 **DIP(의존성 역전 원칙) 위반** 상태였습니다.

---

## 2️⃣ Refactoring Strategy
이번 리팩토링의 핵심 목표는 단 하나입니다.
**"Domain이 저장소 구현, 외부 SDK, API 스펙(DTO) 같은 기술적 세부사항에 의존하지 않는 것**

이를 위해 다음 기준을 세우고 리팩토링을 진행했습니다.
* **추상화:** Domain이 필요로 하는 기능은 Protocol(인터페이스)로 정의한다.
* **구현 격리:** 실제 저장 방식이나 SDK 연동 등 세부 구현은 Data 계층으로 밀어낸다.
* **책임 위임:** DTO 생성/변환 책임은 Data 계층(Repository, Mapper)으로 이동시킨다.

> **💡 참고 (Trade-off):** FeelTalk는 프로젝트 전반이 RxSwift 기반입니다. 엄밀한 의미의 순수 Domain을 위해선 Rx 의존성도 걷어내야 하지만, 비동기 스트림 처리의 일관성을 위해 이번 리팩토링에서는 비동기 타입(`Observable`, `Single`)을 Domain에 유지하고, 인프라 및 계약 의존성 제거에 집중했습니다.

---

## 3️⃣ Refactoring

### 3-1. 인프라 의존성 분리 (저장소 / 외부 SDK)

가장 먼저 Domain에 '어떻게'가 아닌 '무엇을' 해야 하는지 명시하는 추상화된 경계(bondry)를 만들었습니다.

#### Step 01. Domain에 경계(Protocol) 정의

UseCase는 이제 기술적 세부사항(Keychain, Firebase)을 모른 채, "토큰 저장/조회가 필요하다", "푸시 토큰이 필요하다"는 정책만 알게 됩니다.

```swift
// file: "Domain/Interface/Auth/AuthTokenStore.swift"
protocol AuthTokenStore {
    @discardableResult
    func saveAccessToken(_ token: String) -> Bool
    
    @discardableResult
    func saveRefreshToken(_ token: String) -> Bool
    // ...
}
```
```swift
// file: "Domain/Interface/Auth/PushTokenProvider.swift"
protocol PushTokenProvider {
    func fetchToken() -> Single<String?>
    func deleteToken() -> Single<Void>
}
```

#### Step 02. Data 계층에서 구현체 제공

Domain에서 정의한 Protocol을 Data 계층에서 구체적인 기술(Keychain, Firebase)을 사용해 구현합니다.

```swift
// file: "Data/Local/Auth/KeychainAuthTokenStore.swift"
final class KeychainAuthTokenStore: AuthTokenStore {
    func saveAccessToken(_ token: String) -> Bool { KeychainRepository.addItem(value: "accessToken", key: token) }
    // ...
}
```

```swift
// file: "Data/Push/FirebasePushTokenProvider.swift"
import RxSwift
import FirebaseMessaging

final class FirebasePushTokenProvider: PushTokenProvider {
    func fetchToken() -> Single<String?> {
        Single.create { single in
            let token = Messaging.messaging().fcmToken
            single(.success(token))
            
            return Disposables.create()
        }
    }
    // ...
}
```

#### Step 03. UseCase에서 직접 의존 제거 및 주입

기존 Login / SignUp UseCase에 존재하던 import FirebaseMessaging과 KeychainRepository 직접 호출 코드를 모두 제거했습니다. 대신, 생성자를 통해 외부에서 구현체를 주입받도록 변경했습니다.

```swift
// file: "Domain/UseCase/SignUp/SignUpUseCase.swift"
final class DefaultSignUpUseCase: SignUpUseCase {
    private let signUpRepository: SignUpRepository
    private let tokenStore: AuthTokenStore
    private let pushTokenProvider: PushTokenProvider

    // 구체 클래스가 아닌 Protocol에 의존
    init(signUpRepository: SignUpRepository, 
         tokenStore: AuthTokenStore, 
         pushTokenProvider: PushTokenProvider) {
        self.signUpRepository = signUpRepository
        self.tokenStore = tokenStore
        self.pushTokenProvider = pushTokenProvider
    }
}
```

이제 Presentation(Coordinator) 단계에서 UseCase를 초기화할 때, Data 계층의 구현체들을 조립하여 주입해 줍니다.

```swift
// file: "Presentation/SignUp/Coordinator/DefaultSignUpCoordinator.swift"
final class DefaultSignUpCoordinator: SignUpCoordinator {
    func start() {
        self.signUpViewController.viewModel = SignUpViewModel(
            coordinator: self,
            signUpUseCase: DefaultSignUpUseCase(
                signUpRepository: DefaultSignUpRepository(),
                tokenStore: KeychainAuthTokenStore(),
                pushTokenProvider: FirebasePushTokenProvider()))
            
        // ...
    }
}
```

#### 3-2. API 스펙 의존성 분리 (DTO / Mapper)

다음으로, API의 변경이 Domain에 영향을 미치지 않도록 DTO 의존성을 제거했습니다.

#### Step 04. DTO 매핑 책임을 Data 계층(Mapper)으로 이동

Domain Entity 내부에 있던 RequestDTO 변환 로직을 삭제하고, Data 계층에 전용 Mapper를 생성하여 책임을 넘겼습니다.

```swift
// file: "Data/Network/Mapper/Auth/UserAuthInfoMapper.swift"
enum UserAuthInfoMapper {
    static func toAuthNumberRequestDTO(from entity: UserAuthInfo) -> AuthNumberRequestDTO? {
        // Entity 데이터를 기반으로 API 스펙에 맞는 DTO 생성
    }
}
```

#### Step 05. RequestDTO 생성 책임을 Repository로 이동

UseCase가 RequestDTO를 직접 생성해 전달하던 구조를 개선했습니다. Domain은 순수한 Entity와 필요한 데이터(토큰 등)만 넘기고, 네트워크 요청 직전 DTO를 조립하는 역할은 Data 계층인 Repository가 수행합니다.

```swift
// file: "Domain/Interface/Repository/SignUp/SignUpRepository.swift"
// Before: Domain이 DTO를 알아야 함
func signUp(_ requestDTO: SignUpRequestDTO) -> Single<Bool>

// After: Domain은 Entity만 전달함
func signUp(_ entity: SignUpInfo, accessToken: String, fcmToken: String) -> Single<Bool>
```

변경된 시그니처에 따라 Repository 구현체가 DTO를 생성합니다.

```swift
// file: "Data/Network/Repository/SignUp/DefaultSignUpRepository.swift"
func signUp(_ entity: SignUpInfo, accessToken: String, fcmToken: String) -> Single<Bool> {
    let requestDTO = SignUpRequestDTO(
        accessToken: accessToken,
        nickname: entity.nickname,
        marketingConsent: entity.marketingConsent,
        fcmToken: fcmToken
    )

    return Single.create { observer in
        // 통신 로직 진행...
    }
}
```

---

## 🎯 Result

![Thumbnail](/assets/img/blog/ios/feelTalk_clean_architecture_refactoring_image05.png)

이번 리팩토링을 통해 계층별 역할이 명확해지고, 의존성 방향이 완벽하게 통제되었습니다. 그 결과 다음과 같은 이점을 얻을 수 있었습니다.

1. **테스트 용이성 (Testability) 향상**
   이전에는 UseCase를 테스트하려면 실제 Keychain 환경이나 Firebase 초기화가 필요했습니다. 이제는 `AuthTokenStore`나 `PushTokenProvider`의 Mock(가짜) 객체를 주입하여 **순수 비즈니스 로직만 독립적으로, 그리고 매우 빠르게 단위 테스트(Unit Test)**할 수 있게 되었습니다.

2. **인프라 교체의 유연성**
   만약 추후 푸시 알림 서비스를 Firebase에서 다른 솔루션으로 교체하거나, 토큰 저장 방식을 변경하더라도 **Domain 계층의 코드는 수정할 필요가 없습니다.** Data 계층에 새로운 구현체를 만들고 갈아 끼우기만 하면 됩니다.

3. **API 스펙 변경으로부터의 격리**
   서버 API 명세가 바뀌어 Request DTO 구조가 변경되더라도, 이는 Data 계층(Repository, Mapper) 내에서만 처리됩니다. 앱의 핵심 정책인 Domain Entity와 UseCase는 외부 계약 변화에 흔들리지 않습니다.

---

## 💡 회고 및 마무리

이번 리팩토링을 진행하며 **"계층(폴더)을 나눈다고 해서 Clean Architecture가 완성되는 것이 아니라, 의존성의 방향을 통제하는 것이 본질이다"**라는 사실을 깊이 체감할 수 있었습니다.

물론 완벽한 순수 Swift 형태의 Domain을 구축하지 못하고 RxSwift에 의존성을 남겨둔 타협점(Trade-off)은 아쉬움으로 남습니다. 하지만 당장의 비동기 처리 생산성을 확보하면서 가장 치명적이었던 인프라/계약 의존성을 끊어냈다는 점에서 충분히 의미 있는 작업이었습니다. 

향후 앱 고도화 과정에서 Swift Concurrency(`async/await`)를 도입하게 된다면 Domain에서 외부 프레임워크를 완전히 걷어낼 수 있을 거라 기대합니다. 이번 Auth 도메인에서 정립한 의존성 분리 기준을 발판 삼아, 다른 도메인 영역으로도 이 구조를 점진적으로 확장해 나갈 계획입니다.
---
layout: post
title: "Book_Kitty - Coordinator(01)"
sitemap: false
categories: [study, ios]
tags: [ios]
---

* toc
{:toc}

## ✋ Introduce
기존 MVVM 패턴에서는 ViewController가 화면 전환(Navigatoin) 로직을 담당하게 되면서 다음과 같은 문제가 발생할 가능성이 있었습니다.

1. <span style="color:red">단일 책임 원칙(SRP) 위반</span> ➔ 화면 전환 로직이 ViewController에 포함되며, UI처리와 화면 전환이라는 두 가지 책임을 가지게 됨.
2. <span style="color:red">높은 결합도</span> ➔ ViewController가 직접적으로 화면 전환을 처리하면, 의존성이 강해져 테스트가 어렵고 재사용성이 낮아짐.

<br>

아래 코드는 ViewController에서 보편적으로 사용되는 화면 전환 로직입니다.
```swift
class HomeViewController: UIViewController {
  // ...

  func navigateToDetail() {
    let detailVC = DetailViewController()
    navigationController?.pushViewController(detailVC, animated: true)
  }
}
```
해당 코드를 살펴보면 HomeViewController가 DetailViewController를 직접 생성하고 네비게이션 로직을 수행하고 있습니다. 이때 HomeViewController와 DetailViewController 간 강한 결합이 발생합니다.

또한 DeatilViewController가 변경되면 HomeViewController의 코드도 수정해야 하므로 유지 보수성이 떨어지고, 다른 화면으로 이동해야 할 경우, navigateToDetail() 메서드를 계속해서 수정해야 합니다.

## 🤔 Why?
단순한 화면 전환 로직을 위해 Coordinator를 추가로 구현해야 하는 부담, 그리고 이를 처음 접하는 팀원들의 러닝 커브 등 몇 가지 문제가 있었습니다.

하지만, MVVM만으로 프로젝트를 진행했을 때 프로젝트 규모가 커질수록 ViewController의 역할이 비대해지고, 화면 이동 흐름을 체계적으로 관리하기 여려워질 가능성이 높았습니다. 

당장의 작은 불편함 때문에 유지보수성과 확장성을 포기할 수 없었기 때문에, **화면 전환을 보다 일관되고 체계적으로 관리하며, ViewController간 결합도를 낮추면서도 유연한 설계를 지원하는 Coordinator 패턴을 도입하게 되었습니다.**

## 📐설계
<div style="text-align: center;">
  <img src="/assets/img/blog/ios/coordinator03.png" alt="Memory-structure image" loading="lazy" />
</div>
위 그림은 Book_Kitty 프로젝트의 MVVM 구조에서 Coordinator를 적용한 MVVM-C 아키텍처 모식도입니다.

* View는 ViewModel에 의존하며, View에서 발생한 사용자 액션은 ViewModel의 Input 스트림을 통해 ViewModel 내에서 방출됩니다.

* Coordinator는 화면 이동을 관리하기 위해 ViewModel에 의존하여 View로부터 전달되서 ViewModel에서 방출된 사용자 이벤트를 바인딩 합니다.
  * BookKitty 프로젝트에서는 **"RxSwift 기반의 데이터 스트림을 활용하여, ViewModel이 Coordinator에 직접 의존하지 않도록 설계했습니다".**
  * 대신, ViewModel에서 **화면 전환 이벤트를 방출하는 Relay를 선언하고 Coordinator가 이 Relay를 구독**하여 화면 전환을 처리하는 구조로 설계했습니다.
  * 위와같은 설계를 통해 ViewModel과 Coordinator의 결합도는 감소시키고 RxSwift 데이터 스트림의 자연스러운 연결을 구현했습니다.

## 📷 프로토콜
Coordinator 프로토콜을 정의한 이유는 일관된 아키텍처를 유지하면서 화면 전환 로직을 분리하고, 유연한 확장성을 제공하기 위함입니다. 이를 통해 각 Coordinator가 동일한 방식으로 동작하도록 보장하고, 새로운 화면 흐름이 필요할 때 쉽게 추가할 수 있습니다. 

```swift
protocol Coordinator: AnyObject {
    // ...

    var finishDelegate: CoordinatorFinishDelegate? { get set }

    var childCoordinators: [Coordinator] { get set }

    func start()
    func finish()
}
``` 

### childCoordinators
Coordinator 패턴을 활용하다 보면, 하나의 화면 흐름만 관리하는 것이 아니라 여러 개의 화면 흐름을 관리해야 할 때가 있습니다. 이때, 부모 Coordinator가 자식 Coordinator를 관리하는 구조가 필요한데, 이를 위해 childCoordinators를 활용했습니다.

앱이 커질수록 단일 Coordinator로 모든 화면 전환을 관리하는 것을 비효율적입니다. 따라서 **각 흐름을 담당하는 Coordinator를 별도로 만들고 부모 Coordinator가 이를 관리하는 방식**이 더 유연하다 판단했습니다. 

이를 위해 부모 Coordinator는 자식 Coordinator를 생성하고, 이를 childCoordintors 배열에 저장하여 관리합니다. 또한, 자식 Coordinator가 종료되면 배열에서 제거하여 메모리 누수를 방지할 수 있습니다.

### func start()
start() 메서드는 Coordinator의 실행을 시작하는 메서드 입니다. 특정 화면을 생성하고, navigationController에 push하여 화면을 표시하는 역할을 합니다.

### func finish() 
```swift
extension Coordinator {
    func finish() {
        childCoordinators.removeAll()
        finishDelegate?.coordinatorDidFinish(childCoordinator: self)
    }
}
```

finish() 메서드는 현재 Coordinator가 종료될 때, 자식 Coordinator를 모두 정리하고 부모에게 종료를 알리는 역할을 합니다.

### finishDelegate
```swift
protocol CoordinatorFinishDelegate: AnyObject {
    func coordinatorDidFinish(childCoordinator: Coordinator)
}
```

finishDelegate는 Coordinator가 자신의 역할을 마친 후, 부모 Coordinator에게 이를 알리는 역할을 수행하는 Delegate입니다. 이를 통해 부모 Coordinator는 특정 Coordinator의 생명 주기가 끝났음을 감지하고 적절한 정리 작업을 수행합니다.

## 마무리
Book_Kitty 프로젝트에서는 MVVM-C 아키텍처와 RxSwift 기반 데이터 스트림을 결합하여 보다 유연하고 확장성 높은 화면 전환 구조를 설계했습니다.

이러한 설계를 통해 대규모 프로젝트에서도 유지보수성과 확장성이 뛰어난 화면 전환 흐름을 구축할 수 있게 되었습니다. 
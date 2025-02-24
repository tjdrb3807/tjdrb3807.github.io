---
layout: post
title: "Book_Kitty - Coordinator(01)"
sitemap: false
categories: [study, ios]
tags: [ios]
---

* toc
{:toc}

## Introduce
프로젝트의 아키텍처 설계 단계에서 MVVM-C 패턴을 도입하여 ViewController의 역할을 단순화하고, 화면 전환 로직을 분리하는 방식을 채택했습니다.    

기존 MVVM 패턴만으로는 화면 전환 책임이 View(ViewController)에 남아 있어 코드 복잡도가 증가하고, 모듈 간 결합도가 높아지는 문제가 발생할 가능성이 있었습니다. 이를 해결하기 위해 Coordinator 패턴을 함께 적용하여 화면 전환 흐름을 별도의 컨포넌트에서 관리하도록 설계했습니다. 

## 설계

본 프로젝트는 3개의 Tab(Home, Question History, My Library)이 존재하며, 플로팅 버튼을 통한 두 개의 플로우(AddBook, AddQuestion)을 제공합니다.   

<div style="text-align: center;">
  <img src="/assets/img/blog/ios/coordinator02.png" alt="Memory-structure image" loading="lazy" />
</div>

<br>
위와같은 화면 전환을 제공하기 위해 TabBarCoordinator로 부터 5개의 ChildCoordinator가 필요하며, 각 코디네이터의 마지막 플로우에 공통적으로 제공하는 BookDetailController 화면 전환은 별도의 Coordinator로 분리하여 재사용성을 확보했습니다.

<div style="text-align: center;">
  <img src="/assets/img/blog/ios/coordinator01.png" alt="Memory-structure image" loading="lazy" />
</div>

## Coordinator 프로토콜 정의
Coordinator 프로토콜을 정의한 이유는 일관된 아키텍처를 유지하면서 화면 전환 로직을 분리하고, 유연한 확장성을 제공하기 위함입니다. 이를 통해 각 Coordinator가 동일한 방식으로 동작하도록 보장하고, 새로운 화면 흐름이 필요할 때 쉽게 추가할 수 있습니다. 

```swift
protocol Coordinator: AnyObject {
    var finishDelegate: CoordinatorFinishDelegate? { get set }

    var childCoordinators: [Coordinator] { get set }

    var navigationController: UINavigationController { get }

    func start()
    func finish()
}

extension Coordinator {
    func finish() {
        childCoordinators.removeAll()
        finishDelegate?.coordinatorDidFinish(childCoordinator: self)
    }
}
```
### finishDelegate: CoordinatorFinishDelegate?
* Coordinator가 자신의 역할을 마쳤을 때, 부모 Coordinator에게 이를 알리는 역할을 수행합니다.
* 화면 흐름이 완료되었을 때 정리 작업을 수행하기 위해 사용합니다.

### childCoordinators: [Coordinator]
* Coordinator는 여러 개의 자식 Coordinator를 가질 수 있습니다.
* 특정 화면에서 새로운 흐름으로 넘어갈 경우, 새로운 Coordinator를 생성하고 childCoordinators 배열에 추가합니다.
* 화면 전환이 완료되면 해당 Coordinator를 제거하여 메모리 누수를 방지합니다.

### navigationController: UINavigationController
* 현재 Coordinator가 관리하는 UINavigationController를 참조합니다.
* 화면을 push/pop하는 역할을 수행하며, 하나의 흐름 내에서 ViewController 간 전환을 담당합니다.

### func start()
* Coordinator의 실행을 시작하는 메서드 입니다.
* 특정 화면을 생성하고, navigationController에 push하여 화면을 표시하는 역할을 합니다.

### func finish() 
* 현재 Coordinator가 종료될 때, 자식 Coordinator를 모두 정리하고 부모에게 종료를 알리는 역할을 합니다.


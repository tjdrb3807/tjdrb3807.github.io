---
layout: post
title: "Book_Kitty - Coordinator(02)"
sitemap: false
categories: [study, ios]
tags: [ios]
---

* toc
{:toc}

## TabBarCoordinator의 역할
TabBarCoordinator는 Coordinator 프로토콜을 구현하며, 앱의 탭 네비게이션을 관리하는 핵심 역할을 합니다.

본 프로젝트는 3개의 Tab(Home, Question History, My Library)이 존재하며, 플로팅 버튼을 통한 두 개의 플로우(AddBook, AddQuestion)을 제공합니다.   

<div style="text-align: center;">
  <img src="/assets/img/blog/ios/coordinator02.png" alt="Memory-structure image" loading="lazy" />
</div>

<br>
위와같은 화면 전환을 제공하기 위해 TabBarCoordinator로 부터 5개의 ChildCoordinator가 필요하며, 각 코디네이터의 마지막 플로우에 공통적으로 제공하는 BookDetailController 화면 전환은 별도의 Coordinator로 분리하여 재사용성을 확보했습니다.

<div style="text-align: center;">
  <img src="/assets/img/blog/ios/coordinator01.png" alt="Memory-structure image" loading="lazy" />
</div>

### Coordinator 패턴 적용
먼저, TabBarCoordinator는 Coordinator 프로토콜을 따르며, 아래의 주요 속성을 가집니다.

```swift
final class TabBarCoordinator: Coordinator {
    weak var finishDelegate: CoordinatorFinishDelegate?
    var childCoordinators: [Coordinator] = []
    var navigationController: UINavigationController
    var tabBarController: TabBarController
    var tabBarViewModel: TabBarViewModel
    private let disposeBag = DisposeBag()
    
    init(_ navigationController: UINavigationController) {
        self.navigationController = navigationController
        tabBarViewModel = TabBarViewModel()
        tabBarController = TabBarController(viewModel: tabBarViewModel)
    }
}
```
* tabBarController를 생성하고 네비게이션 컨트롤러에 추가합니다
* home, qna, book 탭을 관리하는 각각의 Coordinator를 생성하고 childCoordinators에 등록합니다.
* RxSwift를 활용해 이벤트를 감지하고, 특정 액션이 발생하면 화면 전환을 수행합니다.

### start() 메서드 – Coordinator의 시작
start() 메서드는 TabBarCoordinator가 실행될 때 호출되며, 각 탭에 해당하는 Coordinator를 생성하고 tabBarController에 연결하는 핵심 역할을 합니다.

```swift
func start() {
    // 각 탭에 해당하는 Coordinator 생성
    let homeCoordinator = DefaultHomeCoordinator(navigationController)
    let qnaCoordinator = DefaultQuestionCoordinator(navigationController)
    let bookCoordinator = MyLibraryCoordinator(navigationController)

    // childCoordinators에 등록
    addChildCoordinator(homeCoordinator, qnaCoordinator, bookCoordinator)

    // 각 Coordinator 실행
    homeCoordinator.start()
    qnaCoordinator.start()
    bookCoordinator.start()

    // TabBarController에 뷰 컨트롤러 설정
    tabBarController.setViewControllers(
        homeCoordinator.homeViewController,
        qnaCoordinator.questionHistoryViewController,
        bookCoordinator.myLibraryViewController
    )

    // 책 추가 또는 질문 추가 이벤트를 감지하여 적절한 플로우 실행
    tabBarViewModel.navigateToAddBook
        .withUnretained(self)
        .bind(onNext: { owner, _ in
            owner.showAddBookFlow()
        }).disposed(by: disposeBag)

    tabBarViewModel.navigateToAddQuestion
        .withUnretained(self)
        .bind(onNext: { owner, _ in
            owner.showAddQuestionFlow()
        }).disposed(by: disposeBag)

    // 네비게이션 스택에 TabBarController 추가
    navigationController.pushViewController(tabBarController, animated: true)
}
```
* Home, QnA, MyLibrary Coordinator를 생성하고 childCoordinators에 추가합니다.
* 각 Coordinator의 start()를 호출하여 각 탭의 초기화 작업을 수행합니다.
* tabBarController.setViewControllers()를 통해 생성한 뷰 컨트롤러들을 탭바에 등록합니다.
* RxSwift를 활용하여 특정 이벤트(navigateToAddBook, navigateToAddQuestion)가 발생하면 적절한 화면으로 이동합니다.

### 특정 플로우 실행 (showAddBookFlow, showAddQuestionFlow)
책 추가 및 질문 추가와 같은 특정 이벤트 발생 시, 각각의 Coordinator를 실행하는 메서드를 정의합니다.

```swift
private func showAddBookFlow() {
    let addBookCoordinator = AddBookCoordinator(navigationController)
    addChildCoordinator(addBookCoordinator)
    addBookCoordinator.finishDelegate = self
    addBookCoordinator.start()
}

private func showAddQuestionFlow() {
    let addQuestionCoordinator = AddQuestionCoordinator(navigationController)
    addChildCoordinator(addQuestionCoordinator)
    addQuestionCoordinator.finishDelegate = self
    addQuestionCoordinator.start()
}
```

### Coordinator 종료 처리 (coordinatorDidFinish)
Coordinator가 종료되었을 때 childCoordinators에서 제거하고 적절한 탭으로 이동합니다.

```swift
func coordinatorDidFinish(childCoordinator: Coordinator) {
    childCoordinators.removeAll { $0 === childCoordinator }
    if childCoordinator is AddQuestionCoordinator {
        tabBarController.tabBar.selectedIndex.accept(1)
    } else if childCoordinator is AddBookCoordinator {
        tabBarController.tabBar.selectedIndex.accept(2)
    }
    navigationController.popViewController(animated: true)
}
```

##  TabBarController의 역할
TabBarController는 UITabBarController를 상속하지 않고, 커스텀 UIViewController로 구현되었습니다. 이 컨트롤러는 탭 전환 로직과 플로팅 메뉴(Floating Menu)를 관리하는 역할을 합니다.

### setViewControllers(_:) – 탭 관리할 뷰 컨트롤러 설정
```swift
func setViewControllers(_ viewControllers: UIViewController...) {
    self.viewControllers = viewControllers
}
```
* UITabBarController의 setViewControllers(_:)와 유사하게 관리할 뷰 컨트롤러 목록을 저장합니다.
* 가변 매개변수(UIViewController...)를 사용하여 여러 개의 뷰 컨트롤러를 전달받을 수 있도록 구현되었습니다.

###  setupInitialViewController() – 첫 번째 탭을 기본으로 설정
```swift
private func setupInitialViewController() { 
    showViewController(at: 0) 
}
```
* 앱이 실행되었을 때 첫 번째 탭(인덱스 0)을 기본 화면으로 설정합니다.
* 내부적으로 showViewController(at: 0)을 호출하여 첫 번째 화면을 추가합니다.

### showViewController(at:) – 특정 인덱스의 뷰 컨트롤러를 표시
```swift
private func showViewController(at index: Int) {
    guard index >= 0, index < viewControllers.count else {
        return
    }

    let viewController = viewControllers[index]
    addChild(viewController)
    view.addSubview(viewController.view)
    viewController.view.snp.makeConstraints {
        $0.top.horizontalEdges.equalToSuperview()
        $0.bottom.equalTo(tabBar.snp.top)
    }
    viewController.didMove(toParent: self)
    
    [gradientView, tabBar, dimmingView, floatingButton, floatingMenu]
        .forEach { view.bringSubviewToFront($0) }
}
```
* 특정 인덱스(index)의 뷰 컨트롤러를 현재 화면에 추가합니다.

### hideViewController(at:) – 특정 인덱스의 뷰 컨트롤러를 숨김
```swift
private func hideViewController(at index: Int) {
    guard index >= 0, index < viewControllers.count else {
        return
    }

    let viewController = viewControllers[index]
    viewController.willMove(toParent: nil)
    viewController.view.removeFromSuperview()
    viewController.removeFromParent()
}
```
* 특정 인덱스의 뷰 컨트롤러를 화면에서 제거합니다.
* willMove(toParent: nil)을 호출하여 뷰 컨트롤러가 사라질 것임을 알림.
* removeFromSuperview()를 호출하여 화면에서 제거.
* removeFromParent()를 호출하여 부모 뷰 컨트롤러에서 제거.
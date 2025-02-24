---
layout: post
title: "Book_Kitty - BaseViewController"
sitemap: false
categories: [study, ios]
tags: [ios]
---

* toc
{:toc}

## Introduce

Book_Kitty 프로젝트를 설계하면서 유지보수성과 재사용성을 극대화할 수 있는 방법을 고민했습니다. 이를 위해 왜 이러한 원칙이 중요한지 조사한 결과, 다음과 같은 이유를 확인할 수 있었습니다.

* **기능 확장 시 효율성**
  * 최소한의 코드 수정으로 새로운 기능을 추가할 수 있도록 설계하면 개발 속도가 빨라지고, 휴먼 에러 발생 가능성이 줄어듭니다.
* **코드 중복 최소화**
  * 동일한 기능을 여러 곳에서 반복 구현하면 수정할 때마다 모든 곳을 변경해야 하므로 유지보수 부담이 커집니다.
  * 공통 기능을 모듈화하고 재사용 가능한 구조로 만들면, 코드의 일관성을 유지하면서 중복을 최소화할 수 있습니다.
* **가독성과 협업 효율성 향상**
  * 명확한 코드 구조와 컨벤션을 적용하면 가독성이 높아지고, 팀원들이 쉽게 이해하고 수정할 수 있습니다.
  * 여러 개발자가 함께 작업하는 프로젝트에서는 일관된 패턴을 유지하는 것이 중요합니다.

결과적으로 유지보수성과 재사용성을 고려한 설계를 프로젝트에 적용하면 더 적은 노력으로 기능을 확장할 수 있고, 협업이 원활해지며, 코드 품질이 향상됩니다. Book_Kitty 프로젝트에서도 이러한 원칙을 적용하여 장기적으로 관리하기 쉬운 구조를 구축하고자 했습니다.

## Why?
저희 팀은 iOS 개발자 5명으로 구성되어 있으며, 모든 팀원이 UIViewController에서 많은 작업을 수행할 것이라 예상했습니다.

하지만, 다양한 견해를 가진 팀원들이 각자의 스타일대로 UIViewController 내부의 코드를 작성한다면, PR 리뷰 과정이 복잡해지고, 이후 다른 사람의 코드를 리팩토링 시에도 많은 시간이 소요될 것이라 판단했습니다.

이 문제를 해결하기 위해, 저희는 UIViewController를 커스텀한 BaseViewController 구현을 통해 각 ViewController가 BaseViewController를 상속받아 일관된 구조를 유지할 수 있도록 함으로써 코드의 유지보수성과 재사용성을 높이고자 했습니다.

## Class Inheritance vs Protocol
UIViewController 내에서 일관된 코드로 재사용과 유지보수를 확보하는 방법에는 정답이 없습니다. 그중 저희는 클래스 상속과 프로토콜 기반 설계 중 어떤 방식을 선택할지 고민했습니다. 각각의 접근 방식은 장단점이 있으며, 프로젝트 유지보수와 재사용을 고려하여 최적의 방법을 결정해야 했습니다.

### 상속을 선택한 이유
* 코드의 중복을 줄이고 유지보수를 쉽게 하기 위해
  * BaseViewController를 상속하면 **공통 로직을 한 곳에서 관리**할 수 있고, 변경이 필요한 경우 부모 클래스만 수정하면 자동으로 반영이 됩니다.
  * 같은 기능을 여러 번 반복해서 구현할 필요가 없고, 새로운 ViewController를 추가할 때도 코드가 간결해지므로 상속을 채택했습니다.
  * 반면, 프로토콜을 적용하면, 공통 기능을 강제할 수 없다는 단점이 있었습니다.
* 추가적인 기능 확장이 용이
  * BaseViewController에서 기본적인 기능을 제공한 뒤, **"필요에 따라 하위 클래스에서 오버라이드하여 커스텀"**이 가능했습니다.
  * 프로토콜을 채택하면 새로운 기능을 추가할 때 모든 ViewController에서 개별적으로 구현해야 하므로, 코드 중복이 발생할 가능성이 높다고 판단했습니다.
* RxSwift와 함께 사용하기 적합
  * BaseViewController에서는 disposeBag을 제공하여 모든 ViewController에서 일관된 RxSwift 메모리 관리를 보장할 수 있습니다.
  * 프로토콜로 이 기능을 구현하면 각 ViewController마다 DisposeBag을 선언해야 하므로 코드 중복이 발생할 가능성이 높다 판단했습니다.

위와같은 이유로 BaseViewController를 상속하는 방식이 유지보수와 재사용성을 고려한 설계라 판단했습니다.

## 구현
서론이 긴 반면 사실 BaseViewController 구현은 높은 기술을 요구하지 않습니다. 팀 컨벤션에 맞춰 UIViewController에서 필수적으로 구현할 메서드나 프로퍼티를 정의하고 BaseViewController를 상속받아 사용하면 됩니다.

```swift
class BaseViewController: UIViewController {
    let disposeBag = DisposeBag()
    let viewDidLoadRelay = PublishRelay<Void>()

    // MARK: - Lifecycle

    override func viewDidLoad() {
        super.viewDidLoad()
        configureBackground()
        configureHierarchy()
        configureLayout()
        bind()
        viewDidLoadRelay.accept(())
    }

    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)

        recordScreenView()
    }

    /// 뷰 컨트롤러의 배경색을 설정하는 메서드입니다.
    /// 하위 클래스에서 필요에 따라 오버라이드하여 구현합니다.
    func configureBackground() {
        view.backgroundColor = Colors.background0
    }

    /// 뷰 계층 구조를 설정하는 메서드입니다.
    /// 하위 클래스에서 필요에 따라 오버라이드하여 구현합니다.
    func configureHierarchy() {}

    /// 뷰의 레이아웃을 설정하는 메서드입니다.
    /// 하위 클래스에서 필요에 따라 오버라이드하여 구현합니다.
    func configureLayout() {}

    /// RxSwift 바인딩을 설정하는 메서드입니다.
    /// 하위 클래스에서 필요에 따라 오버라이드하여 구현합니다.
    func bindViewModel() {}
}
```
### RxSwift DisposeBag 제공
저희 프로젝트는 MVVM 아키텍처로 설계되어있으며, View와 ViewModel의 바인딩을 RxSwift로 처리합니다. 이를 위해 ViewController에는 메모리 관리를 해주는 disposeBag이 필수적으로 수반되어야 하므로 BaseViewController에 정의했습니다.
<div style="text-align: center;">
  <img src="/assets/img/blog/ios/baseViewController01.png" alt="Memory-structure image" loading="lazy" />
</div>


### 공동 UI설정 메서드 제공
* configureBackground(), configureHierarchy(), configureLayout() 메서드를 제공하여
각 ViewController에서 동일한 구조로 UI를 구성할 수 있도록 합니다.
* 필요에 따라 하위 클래스에서 오버라이드하여 개별적인 설정을 추가할 수 있습니다.
```swift
final class TabBarController: BaseViewController {
    // ...

    override func configureHierarchy() {
        [gradientView, tabBar, dimmingView, floatingButton, floatingMenu]
            .forEach { view.addSubview($0) }
    }

    override func configureLayout() {
        gradientView.snp.makeConstraints {
            $0.top.equalTo(tabBar.snp.top).offset(-Vars.spacing12)
            $0.horizontalEdges.equalToSuperview()
            $0.bottom.equalTo(view.safeAreaLayoutGuide)
        }

        tabBar.snp.makeConstraints {
            $0.leading.equalToSuperview().inset(Vars.paddingReg)
            $0.trailing.equalTo(floatingButton.snp.leading).offset(-Vars.spacing20)
            $0.bottom.equalTo(view.safeAreaLayoutGuide).inset(Vars.spacing4)
            $0.height.equalTo(Vars.viewSizeReg)
        }

        // ...
    }
}
```

## 마무리
Book_Kitty 프로젝트에서 BaseViewController를 설계한 이유는 유지보수성과 재사용성을 극대화하여 개발 속도를 높이고, 코드 일관성을 유지하기 위함입니다.

UIViewController의 상속을 활용하여 공통 로직을 한 곳에서 관리하고, RxSwift와의 결합을 용이하게 만들었으며, 각 ViewController에서 일관된 UI 및 기능을 구성할 수 있도록 했습니다. 이러한 설계를 통해 모든 팀원이 동일한 구조로 개발을 진행할 수 있어 PR 리뷰 효율성 증가, 코드 가독성 향상, 유지보수 부담 감소 등의 효과를 얻을 수 있었습니다.

물론, 프로젝트에 따라 프로토콜 기반의 접근이 더 적합할 수도 있으며, 모든 프로젝트가 동일한 방식을 따라야 하는 것은 아닙니다. 하지만 Book_Kitty의 요구 사항과 팀의 협업 방식을 고려했을 때, BaseViewController를 활용한 접근 방식이 가장 적절하다고 판단했습니다.

앞으로도 팀의 성장과 프로젝트의 확장성을 고려하여 더욱 발전된 구조를 고민하고 개선해 나가겠습니다.
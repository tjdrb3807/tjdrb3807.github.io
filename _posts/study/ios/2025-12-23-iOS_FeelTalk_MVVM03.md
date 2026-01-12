---
layout: post
title: "FeelTalk - MVVM 패턴 리팩토링"
sitemap: false
categories: [study, ios]
tags: [ios]
---

* toc
{:toc}
![Thumbnail](/assets/img/blog/ios/feelTalk_MVVMPatern_refactoring_thumbnail.png)

## ✍️ Introduction
이 글은 [FeelTalk - MVVM 설계 전략 (Input-Output)](https://tjdrb3807.github.io/study/ios/2025-12-23-iOS_FeelTalk_MVVM02/) 기술 문서를 작성하며 코드를 리뷰하는 과정에서,

설계 단계에서 의도했던 MVVM 패턴 구조가, 코드 레벨에서는 명확히 드러나지 않고 있음을 인식하고,

이를 개선하기 위해 진행한 리팩토링 과정을 정리한 글입니다.

---
## 🧱 Background
FeelTalk 프로젝트의 MVVM 패턴 에서 *View*와 *ViewModel*은 다음과 같은 책임에 집중하도록 설계했습니다.
- ***View***: *ViewModel*로 부터 전달받을 값을 UI에 바인딩
- ***ViewModel***: *View*에 필요한 상태를 비즈니스 로직을 통해 생성·제공

하지만 코드를 리뷰하는 과정에서 다음과 같은 구조적 한계가 드러났습니다.

- *ViewModel*의 Output이 *View*에 제공되어야 할 인터페이스가 아닌 내부 구현 방식을 그대로 노출
- 상태(State)와 이벤트(Event)가 모두 Relay로 선언되어 UI 바인딩의 의도가 불명확
- *View*의 UI 바인딩에 `.bind` 메서드가 사용되며, 바인딩 대상의 성격이 코드 레벨에서 드러나지 않음

이를 해결하기 위해, 기능 동작은 그대로 유지한 채 **구조와 표현을 정제하고 추상화 수준을 조정하는 리팩토링**을 진행했습니다.

> 이 글에서는 Answer 도메인을 중심으로 리팩토링을 진행했으며, 정리된 기준은 추후 다른 도메인에도 확장 적용할 예정입니다.

---
## 1️⃣ Output 설계에서 드러난 구조적 문제
```swift
// file: "Code01"
final class AnswerViewModel {
    struct Output {
        let model = PublishRelay<Question>()
        let isActiveAnswerCompletedButton = BehaviorRelay<Bool>(value: false)
        let bottomSheetHiddenObserver = PublishRelay<Void>()
        let popUpAlertObserver = PublishRelay<CustomAlertType>()
        let popUpPressForAnswerToastMessage = PublishRelay<String>()
    }
}
```

기존 Output은 *View*에 제공되어야 할 인터페이스가 아닌, *ViewModel*의 내부 구현 객체(Relay)를 그대로 노출하는 구조였습니다.

이 구조는 다음과 같은 문제를 가지게 됩니다.

- **구현 타입 노출**: *View*가 추상화된 스트림이 아닌 Relay라는 구체적인 구현 타입에 직접 의존
- **캡슐화 파괴**: *View*에서 *ViewModel*의 상태를 임의로 변경할 수 있어, 상태 관리의 주도권이 모호
- **유지보수성 저해**: 내부 구현(Relay)의 변경이 *View*에 직접적인 영향을 미쳐, 계층 간 결합도가 상승

결과적으로 기존 구조는 ***ViewModel*의 Output이 *View*에 무엇을 제공하는가를 표현하기보다 *ViewModel*이 어떻게 상태를 만들어내는가**를 드러내는 데 치중되어 있었습니다.

### 왜 Output에서 구현 타입(Relay)을 노출하면 안되는가?
Relay는 값을 외부로 방출할 뿐만 아니라, 외부에서 값을 주입(`.accept`)할 수 있는 기능을 포함합니다.

이러한 특성은 *ViewModel* 내부의 상태 관리에는 유용하지만, Output으로 노출되는 순간 **"방출되는 결과물"이 아닌 "조작 가능한 구현 객체"**로 전락합니다.

```swift
// file: "Code02"
private func bind(to: viewModel) {
    let output = viewModel.transform(input: input)

    output.model
        .withUnretained(self)
        .bind { vc, model in
            vc.questionTitleView.model.accept(model)
            vc.myAnswerView.model.accept(model)
            vc.partnerAnswerView.model.accept(model)
            vc.setupAnswerCompletedButton(with: model)
        }.disposed(by: disposeBag)    
}
```

위에 작성된 `Code02`는 리팩토링 이전의 코드이며, 구조적인 문제 지점은 다음과 같습니다.
- `output.model`이 Relay 타입이므로, *ViewModel*만 가져야 할 상태 결정권(`.accept`)을 *View*가 가지게 되면서, 데이터 흐름의 단방향 원칙을 깨뜨릴 위험이 있습니다.
- *View*가 추상화된 데이터 흐름(Observable)이 아닌, 주입 가능한 구체 타입(Relay)을 인지하게 됨으로써 *ViewModel*과의 결합도가 높아집니다.

### Output 설계 리팩토링
첫 번째 리팩토링에서는, *ViewModel*의 Output에서 **구현 타입(Relay)을 제거하고, *View*에 제공되어야 할 상태(Interface)만 노출하도록 구조를 변경**했습니다.

리팩토링의 핵심은 다음과 같습니다.

- **읽기 전용 인터페이스 제공**: Output 타입을 `Observable`로 추상화하여 *View*에서의 임의적인 상태 조작을 차단
- **상태 생명주기 제어**: Relay 들을 `transfrom(input:)` 내부 지역변수 선언을 통해 *ViewModel*이 불필요한 상태 저장소로 확장되는 것을 방지하고, Input-Output 관계를 코드 레벨에서 명확하게 드러냄
- **단방향 흐름 강제**: 상태의 생성과 주입은 오직 *ViewModel* 내부 로직에서만 발생하며, *View*는 전달받은 값을 구독하여 화면에 그리는 역할에만 집중

```swift
// file: "Code03"
final class AnswerViewModel {
    // 기존 Output에서 Relay를 직접 노출하던 구조를 제거하고, 
    // View가 읽기 전용으로 파악할 수 있는 스트림 타입(Observable)으로 변경했습니다. 
    struct Output {
        let model: Observable<Question>
        let isActiveAnswerCompletedButton: Observable<Bool>
        let bottomSheetHidden: Observable<Void>
        let popUpAlert: Observable<CustomAlertType>
        let popUpPressForAnswerToastMessage: Observable<String>
    }

    func transform(input: Input) -> Output {
        // 상태의 생명주기를 transform(input:) 호출 범위로 제한함으로써, 
        // ViewModel이 불필요한 상태 저장소로 확장되는 것을 방지하고, 
        // Input–Output 패턴의 의도를 코드 레벨에서 그대로 드러내도록 했습니다.
        let modelRelay = PublishRelay<Question>()
        let isActiveButtonRelay = BehaviorRelay<Bool>(value: false)
        let bottomSheetHiddenRelay = PublishRelay<Void>()
        let popUpAlertRelay = PublishRelay<CustomAlertType>()
        let toastMessageRelay = PublishRelay<String>()

        input.myAnswerObserver
            .asObservable()
            .map { $0 == MyAnswerViewNameSpace.answerInputViewPlaceholder ? false : $0.count > 0 ? true : false }
            .bind(to: isActiveButtonRelay)
            .disposed(by: disposeBag)

        // Output에서는 Relay 자체를 노출하지 않고,
        // View가 구독만 할 수 있는 읽기 전용 스트림으로 변환해 제공합니다.
        return Output(
            model: modelRelay.asObservable(),
            isActiveAnswerCompletedButton: isActiveButtonRelay.asObservable(),
            bottomSheetHidden: bottomSheetHiddenRelay.asObservable(),
            popUpAlert: popUpAlertRelay.asObservable(),
            popUpPressForAnswerToastMessage: toastMessageRelay.asObservable())
    }
}
```

**이 과정을 통해 Output은 상태의 생성 및 관리 방식과 분리되고, *View*는 어떤 상태를 구독해야 하는지만 명확히 알 수 있게 됩니다.**

---
## 2️⃣. 상태(State)와 이벤트(Event)가 구분되지 않은 Output 설계
Output에서 구현 타입을 제거하며 *View*에 제공되는 인터페이스를 정리했지만, 여전히 구조적 문제가 남아 있었습니다.

바로 **지속적인 상태(State)**와 **일회성 이벤트(Event)**가 동일한 스트림 타입으로 표현되고 있는 점입니다.
```swift
// file: "Code04"
final class AnswerViewModel {
    struct Output {
        let model: Observable<Question>
        let isActiveAnswerCompletedButton: Observable<Bool>
        let bottomSheetHidden: Observable<Void>
        let popUpAlert: Observable<CustomAlertType>
        let popUpPressForAnswerToastMessage: Observable<String>
    }
}
```

`Code04`에서 `model`, `isActiveAnswerCompletedButton`는 화면이 유지되는 동안 지**속적으로 관찰되는 상태**에 해당합니다.

반면 `bottomSheetHidden`, `popUpAlert`, `popUpPressForAnswerToastMessage`는 특정 시점에 **한 번 발생하고 소멸되는 이벤트**입니다.

하지만 Output에서는 이 두 성격이 모두 `Observable로` 동일하게 표현되고 있었습니다.

### 상태와 이벤트를 구분하지 않았을 때의 문제
상태와 이벤트가 동일한 타입(`Observable`)으로 표현되면, *View*는 각 스트림이 어떤 성격을 가지는지 코드만으로 판단하기 어렵습니다.

```swift
// file: "Code04"
private func bind(to: viewModel) {
    let output = viewModel.transform(input: input)

    output.model
        .withUnretained(self)
        .bind { vc, model in
            vc.questionTitleView.model.accept(model)
            vc.myAnswerView.model.accept(model)
            vc.partnerAnswerView.model.accept(model)
            vc.setupAnswerCompletedButton(with: model)
        }.disposed(by: disposeBag)    
}
```

위 코드만 보았을 때, `model`이 한 번만 방출되어야 하는 이벤트인지, 상태 변화의 일부인지를 판단하기 어렵게 되면서 *View*는 *ViewModel*의 사용 규칙을 문서나 경험에 의존하게 됩니다.

**즉, *View*는 "어떻게 사용해야 하는 스트림인지"를 암묵적으로 알고 있어야만 올바르게 사용할 수 있는 구조가 됩니다.**

### 상태와 이벤트 분리 리팩토링
두 번째 리팩토링에서는 *View*가 Output 스트림의 성격을 **타입만 보고도 파악할 수 있도록**, 코드 레벨에서 분리하는 방향으로 리팩토링했습니다.

이를 위해 Output 스트림을 `Driver`와 `Signal`로 구분했습니다.

이때 기준이 된 것은, Output의 각 스트림들이 **화면의 상태를 표현하는지**, 아니면 **특정 시점에 한 번 발생하는 이벤트를 전달하는지**였습니다.

상태는 화면이 유지되는 동안 지속적으로 관찰되며, 언제든 현재 값을 기준으로 UI를 다시 그릴 수 있어야 합니다.

따라서 상태를 표현하는 스트림은 다음과 같은 조건을 만족해야 했습니다.
- 항상 **Main Thread**에서 전달될 것
- 에러로 인해 스트림이 종료되지 않을 것
- 화면 재진입 시에도 **최신 상태를 즉시 전달**할 수 있을 것

Driver는 이러한 요구사항을 타입 수준에서 보장합니다. 이로써 *View*는 상태 스트림을 별도의 예외 코드 없이 안전하게 UI에 바인딩할 수 있습니다.

반면 이벤트는 상태와 다르게, 특정 시점에 한 번 방출하고 그 이후에는 의미가 없어지며 Signal은 이러한 이벤트의 성격을 정확히 표현합니다.
- Main Thread에서 전달됨
- 에러를 방출하지 않음
- **구독 시 이전 이벤트를 재전달하지 않음**

이를 통해 *View*는 이벤트를 상태처럼 다루지 않고, 발생 시점에만 처리하도록 자연스럽게 유도됩니다.

이 기준을 바탕으로 Output을 다음과 같이 재설계했습니다.

```swift
final class AnswerViewModel {
    struct Output {
    // 화면이 유지되는 동안 지속적으로 관찰되는 상태
    // - 항상 최신 값을 UI에 반영해야 함
    // - 화면 재진입 시에도 즉시 현재 상태를 전달해야 함
    // - UI 바인딩에 안전해야 함
    let model: Driver<Question>
    let isActiveAnswerCompletedButton: Driver<Bool>

    // 특정 시점에 한 번 발생하고 소멸되는 이벤트
    // - 재구독 시 다시 전달되면 안 됨
    // - 발생 시점에만 UI가 반응해야 함
    let bottomSheetHidden: Signal<Void>
    let popUpAlert: Signal<CustomAlertType>
    let popUpPressForAnswerToastMessage: Signal<String>
    }

    func transform(input: Input) -> Output {
        // 상태(State)는 내부에서 Relay로 생성하지만,
        // Output에서는 Driver로 변환해 읽기 전용으로만 노출합니다.
        let modelRelay = PublishRelay<Question>()
        let isActiveButtonRelay = BehaviorRelay<Bool>(value: false)

        // 이벤트(Event)는 PublishRelay로 발생시키되,
        // Output에서는 Signal로 변환해 일회성 이벤트임을 명확히 드러냅니다.
        let bottomSheetHiddenRelay = PublishRelay<Void>()
        let popUpAlertRelay = PublishRelay<CustomAlertType>()
        let toastMessageRelay = PublishRelay<String>()

        input.myAnswerObserver
            .asObservable()
            .map { $0 == MyAnswerViewNameSpace.answerInputViewPlaceholder ? false : $0.count > 0 ? true : false }
            .bind(to: isActiveButtonRelay)
            .disposed(by: disposeBag)

        return Output(
            model: modelRelay.asDriver(onErrorDriveWith: .empty()),
            isActiveAnswerCompletedButton: isActiveButtonRelay.asDriver(onErrorJustReturn: false),
            bottomSheetHidden: bottomSheetHiddenRelay.asSignal(),
            popUpAlert: popUpAlertRelay.asSignal(),
            popUpPressForAnswerToastMessage: toastMessageRelay.asSignal()
        )
    }
}
```

이 구조를 통해 *View*는 각 Output을 어떻게 화면에 반영해야 하는지, 혹은 어떻게 처리해야 하는지를 **타입만 보고도 명확히 알 수 있게 되었습니다.**

---
## 3️⃣. View에서 `bind` 사용으로 흐려진 책임 경계
앞선 리팩토링을 통해 *ViewModel*의 Output은
- 구현 타입을 숨기고
- 상태와 이벤트를 타입 레벨에서 명확히 구분하는 구조로 개선되었습니다.

하지만 여전히 *View* 코드에서는, 이러한 설계 의도가 충분히 드러나지 않는 문제가 남아 있었습니다.

바로 **상태와 이벤트 모두를 `bind`로 처리하고 있다는 점**입니다.

```swift
output.model
    .withUnretained(self)
    .bind { vc, model in
        vc.questionTitleView.model.accept(model)
        vc.myAnswerView.model.accept(model)
        vc.partnerAnswerView.model.accept(model)
        vc.setupAnswerCompletedButton(with: model)
    }.disposed(by: disposeBag)
```

이 코드는 동작 자체에는 문제가 없지만, *View*의 책임 관점에서는 모호한 부분이 있습니다.

### bind가 만드는 모호함
`bind`는 RxSwift에서 가장 범용적인 바인딩 방식입니다. 그만큼 이 스트림이 어떤 성격을 가지는지를 코드만 보고 판단하기 어렵게 만듭니다.

즉, 위 코드에서는 다음과 같은 정보가 드러나지 않습니다.
- 이 스트림이 상태인지 이벤트인지
- UI 업데이트가 항상 발생해야 하는지, 혹은 한 번만 반응해야 하는지
- Main Thread, 에러 처리 등이 보장되는 스트림인지

이는 앞선 리팩토링을 통해 정리한 상태와 이벤트 구분의 의미를 *View* 레벨에서 약화시키는 요인이 됩니다.

### View의 역할을 코드로 드러내기
상태와 이벤트를 Driver와 Signal로 분리한 이유는, *View*가 판단하지 않고도 올바르게 반응하도록 유도하기 위함이었습니다.

따라서 *View* 역시 이에 맞는 바인딩 방식을 사용해야, 역할 경계가 코드 레벨에서 완성됩니다.

```swift
final class AnswerViewController: Viewcontroller {
    private func bind(to viewModel: AnswerViewModel) {
        let output = viewModel.transform(input: input)

        // model은 화면이 유지되는 동안 지속적으로 관찰되는 상태입니다.
        // Driver를 사용함으로써:
        // - 항상 Main Thread에서 전달되고
        // - 에러로 스트림이 종료되지 않으며
        // - 화면 재진입 시에도 최신 상태를 즉시 전달받을 수 있습니다.
        output.model
            .drive(with: self) { vc, model in
                vc.questionTitleView.model.accept(model)
                vc.myAnswerView.model.accept(model)
                vc.partnerAnswerView.model.accept(model)
                vc.setupAnswerCompletedButton(with: model)
            }.disposed(by: disposeBag)

        // popUpAlert는 특정 시점에 한 번 발생하고 소멸되는 이벤트입니다.
        // Signal을 사용함으로써:
        // - 이전 이벤트가 재전달되지 않고
        // - 발생 시점에만 View가 반응하도록 보장됩니다.
        output.popUpAlert
            .emit(with: self) { vc, alertType in
                guard !vc.view.subviews.contains(where: { $0 is CustomAlertView }) else { return }
                let alertView = CustomAlertView(type: alertType)
                
                alertView.rightButton.rx.tap
                    .map { alertType }
                    .bind { type in
                        tapAlertRightButtonObserver.onNext(type)
                        alertView.hide()
                    }.disposed(by: vc.disposeBag)
                
                vc.view.addSubview(alertView)
                alertView.snp.makeConstraints { $0.edges.equalToSuperview() }
                vc.view.layoutIfNeeded()
                
                alertView.show()
            }.disposed(by: disposeBag)
    }
}
```
---
## 🏁 Conclusion

이번 글에서는 FeelTalk 프로젝트의 `Answer` 도메인을 중심으로, MVVM 패턴 적용 과정에서 드러난 구조적 문제를 점검하고 Output 설계를 리팩토링한 과정을 정리했습니다.

이를 통해 다음과 같은 기준을 확립했습니다.
- *ViewModel*의 Output은 구현 타입이 아닌, *View*에 제공되어야 할 상태와 이벤트만 표현한다
- 지속적으로 관찰되는 값과 일회성 흐름은 상태(State)와 이벤트(Event)로 명확히 구분한다
- 이 구분은 규칙이 아니라, Driver와 Signal 타입 선택을 통해 코드로 강제한다
- *View*는 상태를 해석하지 않고, 타입이 표현하는 의도에 따라 바인딩만 수행한다

**그 결과 *View*와 *ViewModel* 사이의 책임 경계가 분명해졌고, Output은 내부 구현에 의존하지 않는 안정적인 인터페이스로 정리되었습니다.**

이번 리팩토링은 아래 커밋에서 한 번에 정리되었습니다. [AnswerViewModel Output 구조 리팩토링](https://github.com/tjdrb3807/FeelTalk_iOS/commit/c4c5a95d8bfd21b8d921a7e76f63045658ec23dd)

감사합니다.

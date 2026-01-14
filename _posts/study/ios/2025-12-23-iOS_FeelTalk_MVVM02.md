---
layout: post
title: "FeelTalk - MVVM 설계 전략 (Input–Output)"
sitemap: false
categories: [study, ios]
tags: [ios]
---

* toc
{:toc}

![Thumbnail](/assets/img/blog/ios/feelTalk__MVVM 설계 전략_thumbnail.png)

## ✍️ Introduction
이 글은 FeelTalk 프로젝트에서 ***MVVM (Model - View - ViewModel)* 패턴**과 RxSwift를 적용하며, *View*의 책임을 재정의하고 *ViewModel*을 어떤 기준으로 설계했는지를 정리합니다.

이전 글에서는 UIKit 기반 MVC 구조에서 프로젝트 규모가 커지면서 발생했던 ***Massive ViewController* 문제**와, 이를 구조적으로 해결하기 위해 MVVM 패턴을 선택하게 된 배경을 다뤘습니다.

이번 글에서는 그 연장선상에서,

- MVVM 패턴에서 *View*와 *ViewModel*의 책임을 어떻게 분리했는지
- RxSwift를 통해 *View-ViewModel*을 어떤 방식으로 연결했는지

를 중심으로 다루고자 합니다.

> 이 글에서 말하는 *View*는 MVVM 패턴의 개념적 역할을 의미합니다.  
> UIKit 환경에서는 해당 역할을 `UIViewController`가 수행하기 때문에, 본문에서는 *ViewController*를 *View*로 통칭합니다.

---
## 1️⃣. MVVM 패턴에서의 책임 분리
![MVVM pattern diagram](/assets/img/blog/ios/feelTalk_MVVM_diagram.png)

FeelTalk 프로젝트에서 MVVM 패턴은 **UIKit 구조에서 *ViewController*의 책임을 명확히 제한하기 위한 설계 기준**으로 사용되었습니다.

해당 문제 인식과 MVVM 패턴을 선택하게 된 배경에 대해서는 이전 글에서 자세히 다루었습니다.

👉 [FeelTalk - MVVM 도입기(with RxSwift) / Why MVVM?](https://tjdrb3807.github.io/study/ios/2025-12-23-iOS_FeelTalk_MVVM01/#-why-mvvm)

이러한 배경을 바탕으로, FeelTalk 프로젝트에서는 MVVM 패턴을 기준으로 *View*와 *ViewModel*의 책임을 다음과 같이 정의했습니다.

- ***View***
  - UI 구성 및 상태에 따른 화면 반영
  - 사용자 액션을 이벤트 형태로 전달
  - *ViewModel*이 제공하는 상태를 바인딩하여 UI에 반영

- ***ViewModel***
  - *View*로부터 전달받은 이벤트 처리
  - 화면에 필요한 상태 생성
  - 상태를 `Observable` 스트림 형태로 제공

이 구조에서 *View*는 애플리케이션의 상태를 직접 보관하거나 이벤트에 따라 동작을 판단하지 않습니다.

대신 *ViewModel*이 생성한 상태를 구독하고, 이를 화면에 반영하는 역할만 수행합니다.

즉, **"어떤 상태가 필요한가"는 *ViewModel*이 책임이고, "그 상태를 어떻게 보여줄 것인가"는 *View*의 책임**으로 명확히 분리했습니다.

---
## 2️⃣. RxSwift 기반 View-ViewModel 연결 방식 설계
해당 섹션에서는 RxSwift를 어떤 기준으로 MVVM 패턴 설계에 적용했는지,
그리고 그 과정에서 *View*의 책임이 어떻게 정리되었는지를 중심으로 다룹니다.

FeelTalk 프로젝트에 RxSwift를 적용한 배경은 이전 글에서 자세히 다루었습니다.

(👉 [FeelTalk - MVVM 도입기(with RxSwift)](https://tjdrb3807.github.io/study/ios/2025-12-23-iOS_FeelTalk_MVVM01/#-with-rxswift))

* ### RxSwift 적용원칙
    RxSwift를 도입한 목적은 단순히 비동기 처리를 편하게 하기 위함이 아니라, **MVVM 패턴이 의도한 책임 분리가 UIKit 환경에서 일관되게 유지되도록 이를 코드 레벨에서 구조적으로 강제하기 위함**이 핵심 이유였습니다.

    이를 위해 FeelTalk 프로젝트에서는 다음과 같은 원칙을 설정했습니다.

    - *View*가 이벤트 흐름을 판단하지 않도록 할 것
    - 사용자 이벤트와 비동기 상태 흐름을 *View* 외부에서 관리할 것
    - *View*는 오직 바인딩과 UI 반영에만 집중할 것

    이러한 원칙을 바탕으로, 이후 *View*와 *ViewModel* 간의 바인딩은 RxSwift의 이벤트 스트림을 통해 구현했습니다.

* ### *View*에서 RxSwift 역할
    *View*에서는 사용자 액션을 이벤트 스트림으로 변환해 *ViewModel*에 전달하는 책임만 수행하도록 설계했습니다.

    아래 코드는 FeelTalk 프로젝트에서 사용자 액션을 RxSwift 이벤트로 변환해 전달하는 구조를 단순화한 예시입니다.

    ```swift
    // file: "Code01"
    final class AnswerViewController: UIViewController {

        private let viewModel: AnswerViewModel
        private let disposeBag = DisposeBag()

        override func viewDidLoad() {
            super.viewDidLoad()
            bind()
        }

        private func bind() {
            answerTextView.rx.text.orEmpty
                .bind(to: viewModel.answerText)
                .disposed(by: disposeBag)

            viewModel.isSubmitEnabled
                .drive(submitButton.rx.isEnabled)
                .disposed(by: disposeBag)
        }
    }
    ```
    
    * #### 사용자 액션 전달

    `answerTextView`의 텍스트 변경은 사용자 액션에서 발생한 입력입니다. *View*는 해당 입력을 가공하거나 판단하지 않고, *ViewModel*의 `answerText`에 전달합니다.

    이 시점에서 *View*는:
    - 이 텍스트가 유효한지 판단하지 않고
    - 어떤 상태를 만들어야 하는지 알지 못합니다.

    즉, **사용자 액션에 의한 비즈니스 로직 처리는 *ViewModel*의 책임으로 위임합니다**

    * #### 상태 결과 반영

    `isSubmitEnabled`는 여러 입력과 도메인 규칙을 종합해 *ViewModel*에서 계산된 상태의 결과입니다.

    *View*는 해당 값이:
    - 왜 true인지?
    - 어떤 조건에서 false가 되는지?

    알 필요도, 알 수도 없고 계산된 결과(상태)를 그대로 UI 프로퍼티(`submitButton.rx.isEnabled`)에 바인딩할 뿐입니다.

    이 구조를 통해 ***View*는 이벤트 처리와 상태 판단에서 완전히 분리되고, 바인딩만 담당합니다.** 

* ### *ViewModel*에서의 RxSwift 처리 방식
    *ViewModel*은 UI 컴포넌트나 화면 상태를 직접 제어하지 않으며, *View*로 부터 전달된 이벤트를 기반으로 상태를 생성하고, 화면에 필요한 데이터를 제공하는데 집중합니다.

    아래 코드는 `Code01`과 바인딩 된 *ViewModel*이 사용자 입력을 처리하고, 화면에 필요한 상태를 계산하는 책임을 어떻게 수행하는지 보여주는 예시입니다.

    ```swift
    // file: "Code02"
    final class AnswerViewModel {

        let answerText = PublishRelay<String>()
        let isSubmitEnabled = BehaviorRelay<Bool>(value: false)

        private let disposeBag = DisposeBag()

        init() { bind() }

        private func bind() {
            answerText
                .map { !$0.isEmpty }
                .bind(to: isSubmitEnabled)
                .disposed(by: disposeBag)
        }
    }
    ```

    * #### 사용자 입력 스트림

    `answerText`는 *View*에서 전달되는 사용자 입력을 받는 스트림입니다.

    이 값은 단순히 "텍스트가 변경되었다"는 사실만 전달할 뿐, 그 의미는 아직 정의되지 않은 상태입니다.

    *ViewModel*은 이 입력은 기반으로 비즈니스 로직에 따른 해석과 상태 계산을 수행합니다.

    * #### 상태 결과 스트림

    `isSubmitEnabled`는 화면에 필요한 상태를 표현하는 스트림입니다.

    비즈니스 로직을 통해 계산된 결과를 담으며, *View*는 이 값을 그대로 UI에 바인딩합니다.

---
## 3️⃣. 기존 MVVM 패턴의 구조적 한계
위에서 살펴본 MVVM 구조는 *ViewController*의 책임을 제한하고, 비즈니스 로직을 ViewModel로 이동시키는 목적에 충분히 도달했다 판단합니다.

하지만 프로젝트가 확장되면서, 해결하기 어려운 구조적 문제점이 점차 드러났습니다.

### 1. ViewModel의 책임 경계가 흐려지는 문제
초기 ViewModel은 입력 스트림을 직접 프로퍼티로 노출하고, 내부에서 바인딩을 통해 상태를 계산하는 비교적 단순한 구조였습니다.

이 방식은 간단한 화면에서는 문제가 없었지만, 입력이 늘어나면서 다음과 같은 문제가 발생했습니다.
- 어떤 입력이 어떤 상태에 영향을 주는지 코드레벨에서 파악하기 어려움
- ViewModel의 `public` 프로퍼티가 증가
- ViewModel의 사용법이 암묵적으로만 정의

위과 같은 구조는 ViewModel이 "어떤 입력을 받는지"와 "어떤 출력을 제공하는지"가 명시적으로 드러나지 않는 구조가 되기 시작했습니다.

### 2. ViewController와 ViewModel 사이의 계약이 불명확함
기존 구조에서는 ViewController가 ViewModel의 프로퍼티를 직접 참조하며 바인딩을 구성합니다.

이 방식은 편리하지만, 동시에 다음과 같은 문제를 내포합니다.
- ViewController가 ViewModel 내부 구현에 의존하게 됨
- ViewModel의 내부 구조 변경이 ViewController 수정으로 이어짐
- ViewModel이 "어떻게 사용되어야 하는지"가 코드로 강제되지 않음

즉, ViewController와 ViewModel 사이에 명확한 인터페이스가 존재하지 않는 상태였습니다.

---
## 4️⃣. Input–Output 패턴을 도입한 이유
위와 같은 문제를 해결하기 위해 FeelTalk 프로젝트에서는 MVVM 구조 위에 **Input-Output** 패턴을 추가로 적용했습니다.

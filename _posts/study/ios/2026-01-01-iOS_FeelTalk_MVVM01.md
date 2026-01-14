---
layout: post
title: "FeelTalk - MVVM 도입기(with RxSwift)"
sitemap: false
categories: [study, ios]
tags: [ios]
---

* toc
{:toc}

![MVC Massive ViewController 구조](/assets/img/blog/ios/feelTalk_MVVM도입기_thumbnail.png)

## ✍️ Introduction
이 글은 MVVM 패턴을 실제 프로젝트에 적용하며, 아키텍처 설계와 ViewController 책임 분리에 대해 고민했던 과정을 다룹니다.

특히 이전 개인 프로젝트를 진행하며 UIKit 기반 MVC 구조에서 발생했던 Massive ViewController 문제를 통해 아키텍처 설계의 중요성을 인지하게 된 경험을 공유하고자 합니다.

## 🧱 Background
이전 개인 프로젝트에서는 초기 구현 속도를 우선으로 하여 UIKit 기반 MVC 구조를 그대로 사용했습니다.

프로젝트 초반에는 구조적인 문제를 크게 체감하지 못했지만, 화면 규모가 점차 커지고 기능이 추가되면서 ViewController의 역할이 점점 확장되기 시작했습니다.

![MVC Massive ViewController 구조](/assets/img/blog/ios/feelTalk_massive_viewController.png)

그 결과 ViewController는 다음과 같은 책임을 동시에 가지게 되었습니다.
- UI 로직 처리
- 네트워크 요청
- 상태 관리
- 데이터 가공 및 디코딩

아래 코드는 당시 MVC 구조에서 ViewController가 **어떤 책임을 동시에 가지고 있는지**를 보여주는 일부 예시입니다.

🔗 [전체 코드는 GitHub에서 확인할 수 있습니다.](https://github.com/tjdrb3807/SmartMetro_iOS)
```swift
final class MainViewController: UIViewController {
    private var stationCode: Int = 0
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setUp()

        NotificationCenter.default.addObserver(
            self,
            selector: #selector(loadStationDetailViewAndData(_:)),
            name: NSNotification.Name("tapStationOrLineButton"),
            object: nil
        )
    }
}
```
이 시점부터 ViewController는 단순히 화면을 제어하는 역할을 넘어, 애플리케이션의 상태를 보관하고 이벤트 흐름을 직접 제어하는 중심 객체로 동작하게 되었습니다.

또한 네트워크 요청과 데이터 디코딩 로직 역시 ViewController 내부에 직접 포함되어 있었습니다.

아래 코드는 ViewController가 API Endpoint, 네트워크 요청, 데이터 디코딩까지 직접 책임지고 있었던 구조를 보여줍니다.
```swift
private func fetchStationInfoSectionViewData(complitionHandler: @escaping (Result<StationInfoData, Error>) -> Void) {
    let url = "http://localhost:8080/api/v2/stations/\(stationCode)"

    AF.request(url, method: .get)
        .responseData { response in
            switch response.result {
            case let .success(data):
                let result = try JSONDecoder().decode(StationInfoData.self, from: data)
                complitionHandler(.success(result))
            case let .failure(error):
                complitionHandler(.failure(error))
            }
        }
}
```
이 구조에서는 UI 계층과 데이터 계층이 분리되지 않은 상태가 되었습니다.

그 결과 특정 기능을 수정하거나 확장할 때, ViewController 전반을 함께 수정해야 하는 상황이 반복되었습니다.

이로 인해 다음과 같은 문제가 발생했습니다.
- ViewController 수정 시 영향 범위가 넓어짐
- 네트워크 로직과 UI 로직이 강하게 결합
- 테스트 코드 작성 및 유지보수 난이도 증가

이 경험을 통해 **Massive ViewController 문제는 단순한 코드 스타일 문제가 아니라, 책임 분리가 부족한 아키텍처 설계에서 비롯된 구조적 문제**라는 점을 인지하게 되었습니다.

이러한 문제를 반복하지 않기 위해, 이후 진행한 필로우톡 프로젝트에서는 아키텍처 설계 단계에서부터 ViewController의 책임을 명확히 제한하는 것을 핵심 목표로 삼았습니다.

## 🤔 Why MVVM?
Massive ViewController 문제를 해결하기 위한 방법은 MVVM 패턴 이외에도 여러 가지가 있습니다.

MVC 구조를 유지한 채 ViewController 내부 로직을 분리하거나, VIPER, Clean Architecture와 같은 아키텍처를 도입하는 방법도 고려할 수 있습니다.

그러나 필로우톡 프로젝트에서는 다음과 같은 이유로 MVVM 패턴이 가장 적합하다고 판단했습니다.

### 1️⃣. ViewController 책임을 명확히 제한할 수 있는 구조
MVVM 패턴은 ViewController의 역할을 UI 표현과 사용자 이벤트 전달로 명확히 구분할 수 있는 구조를 제공합니다.

이를 통해 비즈니스 로직과 상태 관리를 ViewModel로 자연스럽게 이동시킬 수 있었고, Massive ViewController 문제를 구조적으로 예방할 수 있다고 판단했습니다.

### 2️⃣. 상태 중심 UI와의 조합
필로우톡 프로젝트는 화면 상태 변화와 비동기 이벤트가 빈번한 구조입니다.

MVVM은 ViewModel을 통해 화면 상태를 단일 흐름으로 관리할 수 있어, UI와 상태 간의 관계를 명확하게 표현하는 데 적합하다고 판단했습니다.

이러한 이유를 기반으로 필로우톡 프로젝트에서는 Massive ViewController를 방지하기 위한 대안으로 MVVM 패턴을 선택했습니다.

## 🔗 with RxSwift
MVVM 패턴을 선택한 이후, 자연스럽게 View와 ViewModel을 연결하는 방식에 대한 고민으로 이어졌습니다.

Delegate와 Closure는 UIKit 환경에서 가장 쉽게 적용할 수 있는 바인딩 방식이지만, 화면 복잡도가 증가할수록 다음과 같은 한계가 있었습니다.
- 이벤트와 상태가 분산되어 흐름을 한눈에 파악하기 어려움
- 상태 종류가 늘어날수록 Delegate 메서드 또는 Closure 수가 증가
- ViewController가 상태 분기와 이벤트 처리 로직을 다시 책임지게 됨
- 비동기 이벤트 조합이 복잡해질수록 코드 가독성 저하

이러한 구조에서는 ViewController의 역할을 UI 표현과 이벤트 전달로 명확히 제한하기 어렵다고 판단했습니다.

이러한 문제를 해결하기 위한 대안으로 RxSwift를 선택했습니다.

RxSwift는 다음과 같은 바인딩 방식을 제공합니다.
* 상태와 이벤트를 Observable 스트림으로 통합 관리
* 비동기 이벤트 흐름을 선언형으로 표현
* ViewController의 역할을 바인딩과 UI 업데이트로 명확히 제한
* 상태 변화 흐름을 ViewModel 내부에 집중

이를 통해 ViewController는 비즈니스 로직과 상태 관리에서 분리되고, 초기 목표였던 Massive ViewController 방지에 보다 적합한 구조로 설계할 수 있다고 판단했습니다.

## 🏁 Conclusion
이 글에서는 다음과 같은 설계 판단을 중심으로 정리했습니다.
- MVC 구조에서 발생한 Massive ViewController 문제
- ViewController 책임 분리의 필요성
- MVVM 패턴을 선택한 이유
- RxSwift를 통한 View-ViewModel 바인딩 방식

실제 필로우톡 프로젝트에 MVVM과 RxSwift를 적용한 구현 과정은 아래 글에서 이어서 다루고자 합니다.

👉 [MVVM + RxSwift 실제 적용기](링크)
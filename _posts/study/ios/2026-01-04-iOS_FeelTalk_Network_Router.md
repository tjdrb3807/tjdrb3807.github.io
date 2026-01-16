---
layout: post
title: "FeelTalk - Router 패턴 (with URLRequestConvertible)"
sitemap: false
categories: [study, ios]
tags: [ios]
---

* toc
{:toc}
![Thumbnail](/assets/img/blog/ios/feelTalk_router_thumbnail.png)

## ✍️ Introduction
이 글은 FeelTalk 프로젝트에서 **API 요청을 구성하는 책임을 Router 패턴으로 분리하여 유지보수성과 타입 안정성을 개선한 설계 사례**를 정리한 기술 문서입니다.

Alamofire의 `URLRequestConvertible` 프로토콜을 기반으로 API 명세를 enum으로 구조화하고, API 요청 구성 로직을 **일관되고 예측 가능한 구조**로 정리한 과정을 담고 있습니다.

---
## 🧱 Background
네트워크 통신은 앱 전반에 걸쳐 사용되며, FeelTalk 프로젝트에서는 이를 Repository 계층을 통해 관리하고 있었습니다.

그러나 Repository를 도입한 이후에도 API 요청을 어떻게 구성할 것인지에 대한 기준이 없을 경우 해당 로직이 각 Repository 내부에 분산되는 문제가 발생했습니다.

실제로 FeelTalk 프로젝트 초기에는 다음과 같은 문제가 있었습니다.

- **API 요청 명세의 분산**
  - API 호출이 필요한 Repository마다 URL, HTTP Method, Parameter를 개별 정의
  - 동일한 API 요청 코드가 여러 Repository에 중복
  - API 명세 변경 시 수정 범위가 여러 Repository로 확산

- **타입 안전성 부족**
  - 문자열 기반 Path 관리로 오타를 컴파일 타임에 검출 불가
  - 네트워크 오류가 런타임에서만 드러나는 구조

이러한 문제는 Repository의 역할을 “데이터 접근 추상화”를 넘어 “API 요청 구성”까지 포함하도록 만들었고, 결과적으로 유지보수 비용과 기능 확장 부담을 증가시켰습니다.

이에 따라 FeelTalk 프로젝트에서는 API 요청 구성 책임을 Repository 외부로 분리하고, 이를 전담할 수 있는 별도의 추상화 계층이 필요하다고 판단했습니다.

---
## 🤔 Why Router Pattern?
API 요청 구성 책임을 분리하기 위해, FeelTalk 프로젝트에서는 Router 패턴을 도입했습니다.

Router 패턴은 **API 엔드포인트를 하나의 추상화된 객체로 표현**함으로써 URL, HTTP Method, Header, Parameter 정의를 한 곳에서 관리할 수 있도록 돕습니다.

이를 기반으로, FeelTalk 프로젝트에서는 Router 패턴을 통해 다음과 같은 구조를 목표로 했습니다.

- URL, HTTP Method, Header, Parameter 정의를 한 곳에 집중
- 호출부에서는 "어떤 API를 호출할 것인가"만 표현
- API 요청 구성 책임과 데이터 접근 로직의 역할을 분리

---
## 👀 Alamofire의 URLRequestConvertible
Router 패턴을 적용하기 위해, Alamofire가 네트워크 요청을 처리하는 방식을 먼저 살펴볼 필요가 있습니다.

Alamofire는 모든 네트워크 요청을 `URLRequestConvertible` 프로토콜을 통해 처리합니다.

> Alamofire uses URLRequestConvertible as the foundation of all requests flowing through the request pipeline.

즉, 어떤 객체든 `URLRequest`로 변환할 수 있다면 Alamofire의 요청 파이프라인에 그대로 전달할 수 있습니다.

`URLRequestConvertible`은 단 하나의 메서드인 `asURLRequest()`를 통해 `URLRequest` 구성 과정을 추상화합니다.

![URLReqeustConvertible Code](/assets/img/blog/ios/feelTalk_router_urlRequestConvertible_code.png)

이 구조를 활용하면:
- API 요청 구성 로직을 Router 내부로 캡슐화할 수 있고
- 호출부에서는 네트워크 구현 상세를 알 필요가 없어집니다.

따라서 Router가 `URLRequestConvertible`을 구현하기 위해서는, 각 API 요청이 공통된 형태의 정보(URL, Method, Header, Parameter)를 일관되게 제공할 수 있어야 했습니다.

FeelTalk 프로젝트에서는 이러한 요구사항을 기반으로 Router의 공통 인터페이스를 설계했습니다.

---
## 🛠 Implementation

### 1. Router 공통 인터페이스 정의
Router 간 구현 방식이 달라지는 것을 방지하고, `URLRequestConvertible` 구현에 필요한 정보를 일관되게 제공하기 위해 모든 Router가 반드시 준수해야 하는 공통 프로토콜을 정의했습니다.

```swift
// file: "Code01"
public protocol Router {
    var baseURL: String { get }
    var path: String { get }
    var method: HTTPMethod { get }
    var header: [String: String] { get }
    var parameters: [String: Any]? { get }
    var encoding: ParameterEncoding? { get }
}
```

해당 인터페이스는 각 API 요청이 `URLRequest`로 변환되기 위해 필요한 최소한의 구성 요소를 정의합니다.

이를 통해 도메인이 추가되더라도 API 요청 구성 방식의 일관성을 유지할 수 있도록 설계했습니다.

### 2. 도메인별 API 명세 정의
각 도메인의 API 엔드포인트는 enum으로 정의했습니다.

```swift
// file: "Code02"
enum QuestionAPI {
    case answerQuestion(requestDTO: AnswerQuestionRequestDTO)
    case getLatestQuestionPageNo
    case getQuestion(index: Int)
    case getQuestionList(questionPage: QuestionPage)
    case getTodayQuestion
    case pressForAnswer(requestDTO: PressForAnswerRequestDTO)
}
```

- Why Enum?
  - **타입 안정성 확보**: 문자열 기반의 URL 관리 방식과 달리, **컴파일 타임에 정의된 케이스만 사용할 수 있도록 강제**하여 존재하지 않는 엔드포인트를 호출하는 실수를 사전에 방지합니다.
  - **연관 값을 통한 파라미터 규격화**: `answerQuestion(requestDTO:)`나 `getQuestion(index:)`와 같이 각 API 요청에 필요한 데이터를 연관 값으로 표현함으로써 데이터 누락 없이 안전하게 파라미터를 전달할 수 있습니다.

### 3. URLRequestConvertible 구현
이렇게 enum으로 정의된 각 API 명세는 Router 공통 인터페이스를 만족하도록 구현되며 `URLRequestConvertible`을 통해 실제 네트워크 요청으로 변환됩니다.

```swift
// file: "Code03"
extension QuestionAPI: Router, URLRequestConvertible {
    var baseURL: String { /* Base URL */ }

    var path: String {
        switch self {
        case .answerQuestion:
            return "/..."
        case .getLatestQuestionPageNo:
            return "/..."
        case .getQuestion(let index):
            return "/.../\(index)"
        case .getQuestionList:
            return "/..."
        case .getTodayQuestion:
            return "/..."
        case .pressForAnswer:
            return "/..."
        }
    }

    var method: HTTPMethod {
        switch self {
        case .answerQuestion:
            return .put
        case .getLatestQuestionPageNo, .getQuestion, .getTodayQuestion:
            return .get
        case .getQuestionList, .pressForAnswer:
            return .post
        }
    }

    var header: [String: String] {
        [
            "Content-Type": "application/json",
            "Accept": "application/json"
        ]
    }

    var parameters: [String: Any]? {
        switch self {
        case .answerQuestion(let dto):
            return [
                "index": dto.index,
                "myAnswer": dto.myAnswer
            ]
        case .getQuestionList(let page):
            return ["pageNo": page.pageNo]
        case .pressForAnswer(let dto):
            return ["index": dto.index]
        default:
            return nil
        }
    }

    var encoding: ParameterEncoding? { JSONEncoding.default }

    func asURLRequest() throws -> URLRequest {
        let url = URL(string: baseURL + path)!
        var request = URLRequest(url: url)

        request.method = method
        request.headers = HTTPHeaders(header)

        return try encoding?.encode(request, with: parameters) ?? request
    }
}
```

이 구조를 통해 API 요청은 Router 내부에서 일관되게 정의되며, 호출부에서는 “어떤 API를 사용할 것인가”만 표현할 수 있도록 설계했습니다.

---

## ✅ Result
Router 패턴을 적용한 이후, Background에서 언급했던 **API 요청 명세의 분산 문제는 Router로 중앙화**되었고, 그 결과 Repository는 더 이상 API 요청의 세부 구현을 직접 다루지 않게 되었습니다.

아래는 Router 패턴 적용 이후 FeelTalk 프로젝트의 `DefaultQuestionRepository` 일부 코드입니다.

```swift
// file: "Code04"
AF.request(
    QuestionAPI.getTodayQuestion,
    interceptor: DefaultRequestInterceptor()
)
```

Repository 에서는:
- URL 문자열이나 HTTP Method를 직접 정의하지 않고
- 어떤 API를 호출할 것인지에 대한 의도만 표현합니다.

API 요청에 필요한 모든 명세는 `QuestionAPI` Router 내부에 캡슐화되어 있으며, Repository는 네트워크 요청 결과를 도메인 모델로 변환하는 역할에 집중합니다.

이를 통해 **Repository의 책임은 "API 요청 구성"이 아닌 "데이터 접근 및 응답 매핑"으로 명확히 한정**되었습니다.

결과적으로 Router 패턴 도입 이후:
- API 명세 변경에 따른 수정 범위가 Router로 국한되었고
- Repository 간 중복 코드가 제거되었으며
- 네트워크 레이어 전반의 유지보수성과 가독성이 개선되었습니다


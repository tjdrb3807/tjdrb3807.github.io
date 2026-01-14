---
layout: post
title: "FeelTalk - Router 패턴 (with Alamofire)"
sitemap: false
categories: [study, ios]
tags: [ios]
---

* toc
{:toc}
![Thumbnail](/assets/img/blog/ios/feelTalk_router_thumbnail.png)

## ✍️ Introduction
이 글에서는 FeelTalk 프로젝트의 네트워크 레이어를 구조화하고 유지보수 효율을 높이기 위해 Router 패턴을 도입하게 된 배경과 구체적인 설계 과정을 소개합니다.

특히, Alamofire의 *URLRequestConvertible* 프로토콜을 활용하여 네트워크 요청(Request) 로직을 어떻게 규격화하고 캡슐화했는지를 중점적으로 다루어 보려 합니다.

---
## 🤔 Why Router Pattern?
네트워크 통신은 앱의 핵심 기능이지만, 체계적인 관리 없이 구현하게 되면 비즈니스 로직과 얽혀 스파게티 코드를 만드는 주범이 되고, 이는 곧 프로젝트의 전체적인 유지보수성을 저하시키는 요인이 됩니다.

실제로 FeelTalk 프로젝트를 진행하며 이러한 구조적 관리 부재로 인해 겪었던 실무적인 문제들은 다음과 같습니다.

* **파편화된 네트워크 로직과 중복 코드**

    프로젝트 초기에는 API 호출이 필요한 곳마다 URL 문자열과 파라미터를 직접 작성했습니다. <br>
    하지만 기능이 늘어날수록 동일한 API 호출 코드가 여러 *Repository*에 흩어지게 되었습니다. <br>
    이로 인해 API 명세가 하나라도 변경되면 **프로젝트 전체를 뒤져 수동으로 코드를 수정**해야 하는 유지보수의 비효율성이 발생했습니다.

* **타입 안정성(Type safety) 부재**

    문자열 기반의 경로(Path) 지정은 오타에 매우 취약합니다. <br>
    컴파일 시점에는 오류를 발견할 수 없고, 오직 런타임에 통신 실패를 통해서만 오타를 확인할 수 있다는 점은 개발 안정성을 저해하는 큰 불안 요소였습니다. <br>

이러한 불안 요소를 해결하기 위해, 모든 API 엔드포인트를 enum으로 정의하여 **중앙 집중식으로 관리**하고, **컴파일 타임에 타입 체크**가 가능한 Router 패턴을 FeelTalk 프로젝트에 도입하게 되었습니다.

---
## 👀 Alamofire의 URLRequestConvertible이란?
본격적인 설계에 앞서, Router 패턴의 뼈대가 되는 *URLRequestConvertible* 프로토콜에 대해 이해할 필요가 있습니다.

[Alamofire Official Documents](https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md#urlrequestconvertible)에서는 *URLRequestConvertible*을 다음과 같이 정의하고 있습니다.

> Alamofire uses URLRequestConvertible as the foundation of all requests flowing through the request pipeline.
>
> → Alamofire는 요청 파이프라인을 통해 흐르는 모든 요청의 근간으로 *URLRequestConvertible*을 사용합니다.

즉, 어떤 복잡한 네트워크 요청이라도 *URLRequestConvertible* 프로토콜을 준수하도록 설계하면, Alamofire의 요청 내에서 일관되고 규격화된 방식으로 처리할 수 있음을 의미합니다.

실제 Alamofire 소스 코드를 살펴보면, *URLRequestConvertible* 프로토콜은 단 하나의 메서드인 `asURLRequest()`를 통해 모든 요청을 추상화하고 있습니다.

![URLReqeustConvertible Code](/assets/img/blog/ios/feelTalk_router_urlRequestConvertible_code.png)

이 메서드는 *URLRequestConvertible* 프로토콜이 정의한 필수 요구사항으로, 이를 구현함으로써 어떤 객체든 Alamofire가 요청 파이프라인에 활용할 수 있는 *URLRequest*로 변환됩니다.

내부적으로는 아래 코드처럼 URL 조립, HTTP 메서드 설정, 헤더 주입 등 복잡한 과정을 거치지만, 최종적으로는 완성된 *URLRequest* 하나를 반환하는 구조를 가집니다.

![URLReqeustConvertible Code](/assets/img/blog/ios/feelTalk_router_asURLRequest_code.png)

결과적으로, *URLRequestConvertible* 프로토콜을 활용하면 상세 구현 로직을 내부로 숨기는 완벽한 캡슐화가 가능해집니다.

호출부에서는 구체적인 생성 과정을 몰라도 *URLRequestConvertible*이라는 공통 인터페이스만으로 안전하게 통신을 요청할 수 있게 되는 것입니다.

이러한 캡슐화와 규격화의 원리를 Router 패턴에 적용한다면, 수많은 API 엔드포인트를 체계적으로 관리할 수 있는 강력한 시스템을 구축할 수 있게 됩니다.

---
## 🛠 Implementation
### 1. 중앙 집중식 상수 관리
본격적인 설계에 앞서, 서버 주소와 같은 네트워크 상수들을 체계적으로 관리할 공간이 필요했습니다. 

프로젝트 곳곳에 서버 URL 문자열이 하드코딩되는 것을 방지하기 위해, 별도의 class를 생성하여 **Base URL을 단일 지점에서 관리**하도록 설계했습니다.

```swift
// file: "Code01"
class ClonectAPI {
    static var BASE_URL: String {
        // 실제 주소 대신 예시용 주소를 사용합니다.
        return "https://api.example.com"
    }
}
```

이때, 단순히 값을 묶는 용도라면 enum이 적합할 수 있으나, FeelTalk 프로젝트에서는 향후 서버 환경(Dev, Staging, Prod) 변화에 따른 확장성을 고려했습니다.

상속이 불가능한 enum과 달리, **class는 상속과 다형성을 지원**합니다.

`static var` 기반의 연산 프로퍼티를 사용하여 설계의 가능성을 열어두었으며, 이를 통해 추후 하위 class를 구현하더라도 사용하는 측에서는 부모 타입만을 참조하여 기존 코드의 수정 없이 환경 변화에 유연하게 대응(OCP)할 수 있는 기반을 마련했습니다.

### 2. 도메인별 Router 분리
앞서 구축한 중앙 집중식 상수 관리 체계를 바탕으로, 실제 API 엔드포인트들을 어떻게 구조적으로 배치할 것인 고민했습니다.

모든 API를 하나의 Router에 담게 되면, 관리가 어려울 뿐만 아니라 코드를 수정할 때 마다 해당 파일 전체를 컴파일해야 하는 비효율이 발생합니다. 

이를 방지하기 위해 FeelTalk의 **도메인을 기준으로 다음과 같이 Router를 세분화**하였습니다.

![URLReqeustConvertible Code](/assets/img/blog/ios/feelTalk_router_domain.png)

이렇게 분리된 Router는 각 도메인의 독립성을 보장하며, 특정 기능의 API 명세가 변경되더라도 해당 Router만 수정하면 되는 유지보수의 편의성을 제공합니다.

### 3. 공통 프로토콜 설계
도메인별 Router를 분리하더라도, 각 Router가 서로 다른 방식으로 *URLRequest*를 생성한다면 결국 또 다른 형태의 파편화가 발생하게 됩니다.

이를 방지하기 위해 FeelTalk 프로젝트에서는 **모든 Router가 반드시 준수해야 하는 공통 프로토콜을 정의**했습니다.

```swift
// file: "Code02"
import Alamofire

public protocol Router {
    var baseURL: String { get }
    var path: String { get }
    var method: HTTPMethod { get }
    var header: [String: String] { get }
    var parameters: [String: Any]? { get }
    var encoding: ParameterEncoding? { get }
}
```

모든 네트워크 요청이 (URL, 경로, 메서드, 헤더, 파라미터, 인코딩 방식)이라는 동일한 인터페이스를 갖도록 강제했으며, 이를 통해 도메인이 추가되더라도 네트워크 요청 구조의 일관성을 유지할 수 있습니다.

### 4. 상세 구현: 도메인별 API 정의
설계한 Router 프로토콜을 바탕으로 실제 도메인 로직을 어떻게 구현했는지 살펴보겠습니다. 

먼저, 각 도메인의 엔드포인트들을 enum을 활용해 정의했습니다.

```swift
// file: "Code03"
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
  - **타입 안정성(Type Safety) 확보**: 문자열 기반의 URL 관리 방식과 달리, 컴파일 타임에 정의된 케이스만 사용할 수 있도록 강제하여 존재하지 않는 엔드포인트를 호출하는 실수를 사전에 방지합니다.
  - **연관 값(Associated Value)을 통한 파라미터 규격화**: `answerQuestion(requestDTO:)`나 `getQuestion(index:)`처럼 각 API 요청에 필요한 데이터를 연관 값으로 묶어두었습니다.<br>이를 통해 해당 API 호출 시 필수 데이터를 인터페이스 상에서 명확히 정의할 수 있으며, 데이터 누락 없이 안전하게 파라미터를 전달합니다.

이렇게 enum으로 정의된 각 케이스는 이후 extension 내부의 `switch` 문을 거치며 각기 다른 path, method, parameters로 변환됩니다

```swift
extension QuestionAPI: Router, URLRequestConvertible {
    var baseURL: String { ClonectAPI.BASE_URL }
    
    var path: String {
        switch self {
        case .answerQuestion:
            return "/..."
        case .getLatestQuestionPageNo:
            return "/..."
        case .getQuestion(index: let index):
            return "/.../\(index)"
        case .getQuestionList:
            return "/..."
        case .getTodayQuestion:
            return "/..."
        case .pressForAnswer:
            return "/..."
        }
    }
    
    var method: Alamofire.HTTPMethod {
        switch self {
        case .answerQuestion:
            return .put
        case .getLatestQuestionPageNo, .getQuestion, .getTodayQuestion:
            return .get
        case .getQuestionList, .pressForAnswer:
            return .post
        }
    }
    
    var header: [String : String] {
        switch self {
        case .answerQuestion,
                .getLatestQuestionPageNo,
                .getQuestion,
                .getQuestionList,
                .getTodayQuestion,
                .pressForAnswer:
            return ["Content-Type": "application/json",
                    "Accept": "application/json"]
        }
    }
    
    var parameters: [String : Any]? {
        switch self {
        case .answerQuestion(requestDTO: let requestDTO):
            return ["index": requestDTO.index,
                    "myAnswer": requestDTO.myAnswer]
        case .getLatestQuestionPageNo, .getQuestion, .getTodayQuestion:
            return nil
        case .getQuestionList(questionPage: let questionPage):
            return ["pageNo": questionPage.pageNo]
        case .pressForAnswer(requestDTO: let requestDTO):
            return ["index": requestDTO.index]
        }
    }
    
    var encoding: Alamofire.ParameterEncoding? {
        switch self {
        case .answerQuestion,
                .getLatestQuestionPageNo,
                .getQuestion,
                .getQuestionList,
                .getTodayQuestion,
                .pressForAnswer:
            return JSONEncoding.default
        }
    }
}
```

최종적으로 enum으로 정의된 명세들이 실제 네트워크 요청으로 변환되는 마지막 관문은 `asURLRequest()` 메서드입니다. 

이 메서드는 프로토콜의 요구사항을 준수하며, 분산되어 있던 데이터들을 하나의 URLRequest로 조립합니다.

```swift
// /// URLRequestConvertible 프로토콜의 필수 요구사항 구현 메서드
func asURLRequest() throws -> URLRequest {
    let url = URL(string: baseURL + path)   // 주소(URL) 생성: 서버 주소(baseURL)와 세부 경로(path)를 결합합니다.
    var request = URLRequest(url: url!)     // URLRequest 초기화: 생성된 URL을 바탕으로 네트워크 요청서의 뼈대를 만듭니다.
    
    request.method = method                 // HTTP 메서드 설정: 해당 API에 맞는 전송 방식(GET, POST, PUT, DELETE 등)을 지정합니다.
    request.headers = HTTPHeaders(header)   // HTTP 헤더 주입: JSON 통신 설정 등 미리 정의된 공통 헤더 정보를 요청서에 담습니다.
    
    // 파라미터 인코딩 처리: 
    // 전송할 데이터(parameters)가 있다면, 설정된 인코딩 방식(JSONEncoding 등)에 따라 
    // 데이터를 요청서의 Body에 담거나 URL 뒤에 붙여주는 작업을 수행합니다.
    if let encoding = encoding {
        // 인코딩이 성공하면 파라미터가 포함된 최종 요청서를 반환합니다.
        return try encoding.encode(request, with: parameters)
    }
    
    // 인코딩할 데이터가 없는 경우, 기본 요청서를 그대로 반환합니다.
    return request
}
```

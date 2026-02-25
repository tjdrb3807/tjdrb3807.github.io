---
layout: post
title: "FeelTalk - Base URL 설계 리팩토링"
sitemap: false
categories: [study, ios]
tags: [ios]
---

* toc
{:toc}

## ✍️ Introduction
이 글은 FeelTalk 프로젝트에서 네트워크 요청의 기준이 되는 Base URL을 `static` 프로퍼티로 관리하던 초기 설계를 되돌아보고, **OCP(Open–Closed Principle) 관점에서 이를 리팩토링한 과정**을 정리한 기술 문서입니다.

---
## 🧱 Background
프로젝트 초기 FeelTalk의 네트워크 구조는 모든 API 요청을 단일 서버 대상으로 했으며, 실행 환경에 따른 분기나 서버 교체를 고려할 필요가 없었습니다.

이러한 맥락에서 네트워크 통신에 필요한 `BASE_URL`은 다음과 같이 `static` 프로퍼티로 정의해 사용했습니다.

```swift
// file: "Code01"
final class ClonectAPI {
    static var BASE_URL: String { "https://example.com" }
}
```
당시 이러한 구조는 다음과 같은 이유로 합리적이라 판단했습니다.
- 단일 서버 환경으로 인해 BASE_URL의 변경이 낮음
- 전역으로 접근 가능한 `static` 프로퍼티를 통해 간편하게 참조
- 네트워크 요청 코드 전반에서 중복 없이 동일한 값을 사용할 수 있음

즉, **현재 요구사항을 만족하는 가장 단순한 설계**라는 관점에서 해당 구조를 적용했습니다.

그러나 이후 Router 패턴을 도입하고, API 요청 명세를 구조적으로 분리하는 과정에서 BASE_URL 관리 방식이 갖는 설계적 한계를 인지했습니다.

특히 BASE_URL이 `static` 프로퍼티로 고정되어 있다는 점은 다음과 같은 문제를 초래했습니다.
- 네트워크 실행 환경(Dev/Prod)에 따른 유연한 확장이 불가
- 새로운 요구사항이 추가될 경우 기존 코드를 수정

이러한 한계는 Base URL 관리 방식이 **새로운 요구사항에 닫혀 있고, 확장에 열려 있지 않다는 점**에서 설계 관점의 문제로 이어졌습니다.

특히 실행 환경에 따라 다른 Base URL이 필요해질 경우, 기존 코드를 수정하지 않고는 대응할 수 없다는 점은 OCP(Open–Closed Principle)가 지향하는 방향과 어긋나는 구조였습니다.

<!-- 이러한 인식은 Router 패턴을 기술 문서로 정리하며 전체 네트워크 설계를 되짚는 과정에서 보다 명확해졌습니다. -->


<!-- 당시 FeelTalk 프로젝트는 단일 서버 환경을 사용하고 있었고, 서버 환경 분기(Dev / Prod)에 대한 요구사항도 존재하지 않았기 때문에 해당 구조는 구현 관점에서 충분히 합리적인 선택이었습니다.

실제로 네트워크 요청은 정상적으로 동작했고, Base URL 관리 방식이 즉각적인 문제로 인식되지는 않았습니다.

다만 Router 패턴을 도입한 이후, API 요청 명세를 구조화하고 이를 기술 문서로 설명하는 과정에서 Base URL이 변경 불가능한 전역 상수에 의존하고 있다는 점이 설계 관점에서 어색하게 느껴지기 시작했습니다.

### 1. OCP(Open Closed Principle) 위반
`BASE_URL`이 `static` 프로퍼티로 고정된 구조에서 서버 환경이 추가되거나 변경될 경우 기존 코드 수정이 불가피합니다.

이는 확장 시 코드 수정 필요, 기존 코드에 직접적인 변경 발생 이라는 점에서 확장에는 열려 있고 변경에는 닫혀 있어야 한다는 OCP 원칙을 충족하지 못하는 구조였습니다.

### 2. 네트워크 환경 책임의 모호함
Base URL이 단순 상수 형태로 존재하면서 “현재 어떤 환경의 서버를 사용하는지”, “환경 변경은 어디에서 책임지는지” 가 코드 상에서 명확하게 드러나지 않았습니다.

결과적으로 Base URL은 Router나 Repository 외부에 존재하지만, 명확한 책임 주체 없이 암묵적으로 사용되는 상태였습니다.

---
## 🔧 Refactoring Approach
이러한 한계를 인식한 이후 Base URL을 직접 참조하는 구조에서 벗어나 네트워크 실행 환경을 추상화하는 방향으로 리팩토링을 진행했습니다.

리팩토링의 목표는 다음과 같았습니다.
- Base URL 변경 시 기존 코드 수정 최소화
- 네트워크 환경 책임을 명확히 분리
- Router 및 Repository가 환경 정보에 직접 의존하지 않도록 설계

---
## 🛠 Refactoring Design
### 1. 네트워크 환경 추상화
먼저 네트워크 실행 환경을 추상화하기 위한 프로토콜을 정의했습니다.
```swift
protocol APIenvironment {
    var baseURL: String { get }
}
```

이를 통해 네트워크 레이어는 구체적인 서버 주소를 직접 알 필요 없이, 환경이 제공하는 `baseURL`만 참조하도록 설계했습니다.

### 2. 실행 환경 구현
실제 서버 환경은 APIEnvironment를 채택하는 타입으로 표현했습니다.
```swift
final class ProdAPIEnvironment: APIEnvironment {
    var baseURL: String { "https://example.com" }
}
```

이 구조를 통해 환경 추가 시 기존 코드 수정 없이 확장 가능, baseURL 변경 책임을 환경 객체로 한정 할 수 있도록 했습니다.

### 3. Network Context 구성
네트워크 전반에서 사용할 실행 환경을 하나의 컨텍스트로 묶어 관리했습니다.
```swift
struct NetworkContext {
    let environment: APIEnvironment
}
```

Network Context는 “현재 네트워크 요청이 어떤 환경에서 실행되는가”를 명시적으로 표현하는 역할을 합니다. -->
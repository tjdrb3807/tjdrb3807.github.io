---
layout: post
title: "Swift #Section04 - Protocol"
sitemap: false
image: /assets/img/blog/thumbnail/swift/swift_section04_protocol.png
categories: [study, swift]
tags: [swift]
description: 본 글에서는 Swift의 프로토콜 지향 프로그래밍과 응용을 알아보기 전 프로토콜에 대해 다룬다.
---

* toc
{:toc}

## 🤚 Introduce
프로토콜(Protocol)은 특정 역할을 하기 위한 메서드, 프로퍼티, 기타 요구사항 등의 청사진을 정의한다.
구조체, 클래스, 열거형은 프로토콜을 채택(Adopted)해서 특정 기능을 실행하기 위해 프로토콜의 요구사항을 실제로 구현할 수 있다. ***어떤 프로토콜의 모든 요구사항을 따르는 타입을 '해당 프로토콜을 준수한다(Comform)'고 표현한다.***
타입에서 프로토콜의 요구사항을 충족시키려면 프로토콜이 제시한 모든 기능을 구현해야한다.

## 프로토콜 채택
`protocol` 키워드를 명시해 프로토콜을 정의할 수 있다.
~~~swift
// file: "protocol definition"
protocol ProtocolName {
    // Protocol definition..
}
~~~

구조체, 클래스, 열거형 등에서 프로토콜을 채택하려면 타입 이름 뒤에 콜론(`:`)을 붙여준 후 채택할 프로토콜 이름을 쉼표(`,`)로 구분하여 명시해주며, 이때 클래스가 상위 클래스를 상속받는 하위 클래스라면 상위 클래스 이름 다음에 채택할 프로토콜을 나열한다.
~~~swift
// file: "Code01.swift"
struct SomeStruct: AProtocol, AnotherProtocol {
    // Strcut definition..
}

class SomeClass: SuperClass, AProtocol, AnotherProtocol {
    // Class definition..
}

enum SomeEnum: AProtocol, AnotherProtocol {
    // Enum definition..
}
~~~
## 프로토콜 요구사항
### 프로퍼티 요구
### 메서드 요구
### 가변 메서드 요구
### 이니셜라이저 요구

## 프로토콜의 상속과 클래스 전용 프로토콜
## 프로토콜 조합과 프로토콜 준수 확인
## 프로토콜의 선택적 요구
## 프로토콜 변수와 상수
## 위임을 위한 프로토콜
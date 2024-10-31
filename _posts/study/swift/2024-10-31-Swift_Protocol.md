---
layout: post
title: "Swift Protocol"
sitemap: false
image: /assets/img/blog/thumbnail/swift/swift_instance.png
categories: [study, swift]
tags: [swift]
description: 본 글에서는 Swift 사용자 지정 데이터의 인스턴스 생성 및 소멸에 대해 다룬다.
---

* toc
{:toc}

## 🤚 Introduce
***프로토콜(Protocol)은 특정 역할을 하기 위한 메서드, 프로퍼티, 기타 요구사항 등의 청사진을 정의한다.*** 즉, 프로토콜은 기능을 정의하고 제시할 뿐 스스로 기능을 구현하지 않는다. 

구조체, 클래스, 열거형은 프로토콜을 ***채택(Adopted)***해서 특정 기능을 실행하기 위한 프로토콜 요구사항을 구현하며, 특정 프로토콜의 요구사항을 모두 따르는 타입은 ***'해당 프로토콜을 준수한다(Conform)'***고 표현한다.

---

## 프로토콜 채택
`protocol` 키워드를 사용해서 프로토콜을 정의한다.
~~~swift
// file: "프로토콜 정의.swift"
protocol ProtocolName {
    // Code ...
}
~~~

<br>

구조체, 클래스, 열거형 등에서 프로토콜을 채택하려면 타입 이름 뒤에 `:`을 붙여준 후 채택할 프로토콜 이름을 `,`로 구분해 명시하며, 클래스가 다른 클래스를 상속받는다면 상위 클래스 이름 다음에 채택할 프로토콜을 나열한다.
~~~swift
// file: "프로토콜 채택.swift"
struct SomeStruct: AProtocol, AnotherProtocol {
    // Code ...
}

class SomeClass: SuperClass, AProtocol, AnotherProtocol {
    // Code ...
}

enum SomeEnum: AProtocol, AnotherProtocol {
    // Code ...
}
~~~

---

## 프로토콜 요구사항
프로토콜은 타입이 특정 기능을 실핼하기 위해 필요한 기능을 요구한다. 프로토콜이 자신을 채택한 타입에 요구하는 사항은 프로퍼티나 메서드와 같은 기능이다.

### 프로퍼티 요구
프로토콜은 자신을 채택한 타입이 어떤 프로퍼티를 구현해야 하는지 요구할 수 있다. 이때 프로토콜은 프로퍼티의 종류(저장, 연산)는 별도로 신경쓰지 않으며, 프로퍼티를 읽기 전용 또는 읽기와 쓰기가 모두 가능하게 할지 정한다. 프로토콜을 채택한 타입은 프로토콜이 요구하는 프로퍼티의 이름과 타입만 맞게 구현하면 된다.

프로토콜의 프로퍼티 요구사항은 항상 `var` 키워드를 사용한 변수 프로퍼티로 정의한다. 읽기와 쓰기가 모두 가능한 프로퍼티는 `{ get, set }`이라 명시하며, 읽기 전용은 `{ get }`으로 명시한다.
~~~swift
// file: "Code01.swift"
protocol SomeProtocol {
    var settableProperty: String { get set }
    var notNeedToBeSettableProperty: String { get }
}
~~~

<br>

`class` 타입 프로퍼티와 `static` 타입 프로퍼티를 구분하지 않고 `static` 키워드로 타입 프로퍼티를 요구한다.

~~~swift
// file: "Code02.swift"
protocol AnotherProtocol {
    static var someTypeProperty: Int { get set }
    static var anotherTypeProperty: Int { get }
}
~~~

<br>

프로토콜이 읽고 쓰기가 가능한 프로퍼티를 요구한다면 읽기만 가능한 상수 저장 프로퍼티 또는 읽기 전용 연산 프로퍼티를 구현할 수 없다. 프로토콜이 읽기 전용 프로퍼티를 요구한다면 상수 저장 프로퍼티나 읽기 전용 연산 프로퍼티를 포함해서 어떤 식으로든 프로퍼티를 구현할 수 있다. 쓰기만 가능한 프로퍼티는 없으니 타입에 구현해주는 프로퍼티는 무엇이든 상관없다.

~~~swift
// file: "Code03.swift"
protocol Sendable {
    var from: String { get }
    var to: String { get }
}

class Message: Sendable {
    var sender: String
    var from: String { self.sender }    // (읽기 전용) 연산 프로퍼티
    var to: String
    
    init(sender: String, receiver: String) {
        self.sender = sender
        self.to = receiver
    }
}

class Mail: Sendable {
    var from: String                    // 저장 프로퍼티
    var to: String
    
    init(sender: String, receiver: String) {
        self.from = sender
        self.to = receiver
    }
}
~~~

### 메서드 요구
프로토콜은 특정 인스턴스 메서드나 타입 메서드를 요구할 수 있으며, 메서드의 실제 구현부 (`{ }`)는 제외하고 이름, 매개변수, 반환타입만 작성한다.

프로토콜의 메서드 요구에서는 매개변수 기본값을 지정할 수 없으며, 타입메서드를 요구할 때는 `static` 키워드를 명시한다. 요구한 타입 메서드를 클래스에서 구현할 때 `static` 키워드나 `class` 키워드 어느 쪽을 사용해도 무방하다.

~~~swift
// file: "Code04.swift"
protocol Receiveable {
    func received(data: Any, from: Sendable)
}

protocol Sendable {
    var from: Sendable { get }
    var to: Receiveable? { get }
    
    func send(data: Any)
    
    static func isSendableInstance(_ instance: Any) -> Bool
}

// 수신, 발신이 가능한 Message 클래스
class Message: Sendable, Receiveable {
    var from: Sendable { self }
    var to: Receiveable?
    
    func send(data: Any) {
        guard let receiver = self.to else {
            print("Message has no receiver")
            
            return
        }
        receiver.received(data: data, from: self.from)
    }
    
    func received(data: Any, from: Sendable) {
        print("Message received \(data) from \(from)")
    }
    
    class func isSendableInstance(_ instance: Any) -> Bool {
        if let sendableInstance = instance as? Sendable {
            return sendableInstance.to != nil
        }
        
        return false
    }
}

class Mail: Sendable, Receiveable {
    var from: Sendable { self }
    var to: Receiveable?
    
    func send(data: Any) {
        guard let receiver = self.to else {
            print("Mail has no receiver")
            
            return
        }
        
        receiver.received(data: data, from: self.from)
    }
    
    func received(data: Any, from: any Sendable) {
        print("Mail received \(data) from \(from)")
    }
    
    static func isSendableInstance(_ instance: Any) -> Bool {
        if let sendableInstance = instance as? Sendable {
            return sendableInstance.to != nil
        }
        
        return false
    }
}

let myPhoneMessage = Message()
let yourPhoneMessage = Message()

myPhoneMessage.send(data: "Hello")  // Message has no receiver

myPhoneMessage.to = yourPhoneMessage
myPhoneMessage.send(data: "Hello")  // Message received Hello from Message

let myMail = Mail()
let yourMail = Mail()

myMail.send(data: "Hi")             // Mail has no receiver

myMail.to = yourMail
myMail.send(data: "Hi")             // Mail received Hi from Mail

myMail.to = myPhoneMessage
myMail.send(data: "Bye")            // Massage received By from Mail

Message.isSendableInstance("Hello")             // false
Message.isSendableInstance(myPhoneMessage)      // true
Message.isSendableInstance(yourPhoneMessage)    // false

Mail.isSendableInstance(myPhoneMessage)         // true
Mail.isSendableInstance(myMail)                 // true
~~~


### 가변 메서드 요구

### 이니셜라이저 요구

## 프로토콜의 상속과 클래스 전용 프로토콜

## 프로토콜 조합과 프로토콜 준수 확인

## 프로토콜의 선택적 요구

## 프로토콜 변수와 상수

## 위임을 위한 프로토콜

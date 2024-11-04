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
프로토콜은 타입이 특정 기능을 실행하기 위해 필요한 기능을 요구한다. 프로토콜이 자신을 채택한 타입에 요구하는 사항은 프로퍼티나 메서드와 같은 기능이다.

### 프로퍼티 요구
프로토콜은 자신을 채택한 타입이 어떤 프로퍼티를 구현해야 하는지 요구할 수 있다.
***프로토콜을 책택한 타입은 프로토콜이 요구하는 프로퍼티의 이름과 타입만 맞도록 구현해주면 된다.*** 따라서 프로토콜은 요구할 프로퍼티의 종류(연산 프로퍼티, 저장 프로퍼티)는 별도로 신경쓰지 않는다. 
다만 프로퍼티를 읽기 전용으로 할지 읽고 쓰기가 모두 가능하게 할지는 프로토콜이 정한다.

프로토콜이 읽고 쓰기가 가능한 프로퍼티를 요구한다면 읽기만 가능한 상수 저장 프로퍼티나 읽기 전용 연산 프로퍼티로는 구현할 수 없다.
만약 프로토콜이 읽기 전용 프로퍼티를 요구한다면 타입에 프로퍼티를 구현할 때 상수 저장 프로퍼티나 읽기 전용 연산 프로퍼티를 포함해서 어떤 식으로든 프로퍼티를 구현할 수 있다.

프로토콜의 프로퍼티 요구사항은 항상 `var` 키워드를 명시한 변수 프로퍼티로 정의한다 읽고 쓰기가 모두 가능한 프로퍼티는 프로퍼티 정의 뒤에 `{ get set }`이라 명시하고, 읽기 전용 프로퍼티는 `{ get }`이라 명시한다.
~~~swift
// file: "Code02.swift"
protocol SomeProtocol {
    var settableProperty: String { get set }
    var notNeedToBeSettableProperty: String { get }
}

protocol AnotherProtocol {
    static var someTypeProperty: Int { get set }
    static var anotherTypeProperty: Int { get }
}
~~~

타입 프로퍼티를 요구하기 위해 `static` 키워드를 사용하며, 상속 가능 여부를 구분하지 않고 모두 `static` 키워드를 명시해 타입 프로퍼티를 요구한다.

~~~swift
// file: "Code03.swift"
protocol Sendable {
    var from: String { get }
    var to: String { get }
}

class Message: Sendable {
    var sender: String
    var from: String { sender }
    var to: String
    
    init(sender: String, receiver: String) {
        self.sender = sender
        self.to = receiver
    }
}


class Mail: Sendable {
    var from: String
    var to: String
    
    init(sender: String, receiver: String) {
        self.from = sender
        self.to = receiver
    }
}
~~~

Code03에서 `Message` 클래스의 `from` 프로퍼티는 읽기 전용 연산 프로퍼티, `Mail` 클래스의 `from` 프로퍼티는 저장 프로퍼티로 구현된 것을 확인할 수 있다.
`Sendable` 프로토콜에서 요구한 프로퍼티는 읽기 전용 프로퍼티였지만 실제 프로토콜을 채택한 클래스에서 구현할 때는 읽고 쓰기가 가능한 프로퍼티로 구현할 수 있다.

### 메서드 요구
프로토콜은 특정 인스턴스 메서드나 타입 메서드를 요구할 수 있다.
프로토콜에서 메서드를 정의할 때 실제 구현부(`{ }`)를 제외하고 메서드 이름과 매개변수, 반환 타입만 명시하며, 가변 매개변수도 허용한다.

프로토콜의 메서드 요구에서는 매개변수 기본값을 지정할 수 없다.
타입 메서드를 요구할 때는 `static` 키워드를 명시하며, `static`키워드를 사용해 요구한 타입 메서드를 클래스에서 실제로 구현할 때는 `static` 키워드나 `class` 키워드 어느 쪽을 사용해도 무방하다.
~~~swift
// file: "Code04.swift"
// 무언가를 수신받을 수 있는 기능 요구
protocol Receiveable {
    func received(data: Any, from: Sendable)
}

// 무언가를 발신할 수 있는 기능 요구
protocol Sendable {
    var from: Sendable { get }
    var to: Receiveable? { get }
    
    func send(data: Any)
    
    static func isSendableInstance(_ instance: Any) -> Bool
}

// 수신, 발신 가능한 Message 클래스
class Message: Sendable, Receiveable {
    // 발신은 발신 가능 객체, 즉 Sendable 프로토콜을 준수하는 타입의 인스턴스여야 한다.
    var from: Sendable { self }
    // 상대방은 수신 가능한 객체, 즉 Receiveable 프로토콜을 준수하는 타입의 인스턴스여야 한다.
    var to: Receiveable?
    
    // 메세지 발신
    func send(data: Any) {
        guard let receiver = self.to else {
            print("Message has no receiver")
            return
        }
        
        // 수신 가능한 인스턴스의 received 메서드 호출
        receiver.received(data: data, from: self.from)
    }
    
    // 메세지 수신
    func received(data: Any, from: any Sendable) { print("Message received \(data) from \(from)") }
    
    // class 메서드이므로 상속 가능
    class func isSendableInstance(_ instance: Any) -> Bool {
        if let sendableInstance = instance as? Sendable { sendableInstance.to != nil }
        
        return false
    }
}

// 수신, 발신이 가능한 Mail 클래스
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
    
    func received(data: Any, from: any Sendable) { print("Mail received \(data) from \(from)") }
    
    // static 메서드이므로 상속 불가능
    static func isSendableInstance(_ instance: Any) -> Bool {
        if let sendableInstance = instance as? Sendable { sendableInstance.to != nil }
        
        return false
    }
}

let myPhoneMessage = Message()
let yourPhoneMessage = Message()

// 아직 수신받을 인스턴스가 존재하지 않음.
myPhoneMessage.send(data: "Hello")  // Message has no receiver
// Message 인스턴스를 발신과 수신이 모두 가능하므로 메세지를 주고 받을 수 있다.
myPhoneMessage.to = yourPhoneMessage
myPhoneMessage.send(data: "Hello")  // Message received Hello from Message

let myMail = Mail()
let yourMail = Mail()

myMail.send(data: "Hi")     // Mail has no receiver
myMail.to = yourMail
myMail.send(data: "Hi")     // Mail received Hi from Mail

myMail.to = myPhoneMessage
myMail.send(data: "Bye")    // Message received Bye from Mail

// String은 Sendable 프로토콜을 준수하지 않음.
Message.isSendableInstance("Hello")             // false
// Mail와 Message는 Sendable 프로토콜을 준수.
Message.isSendableInstance(myPhoneMessage)      // true
// yourPhoneMessage는 to 프로퍼티가 설정되지 않아 보낼 수 없는 상태
Message.isSendableInstance(yourPhoneMessage)    // false

Mail.isSendableInstance(myPhoneMessage)         // true
Mail.isSendableInstance(myMail)                 // true
~~~

> **NOTE_타입으로서 프로토콜**
>
> 프로토콜은 요구만 하고 스스로 기능을 구현하지 않는다. 
> 그렇지만 프로토콜은 코드에서 완전한 하나의 타입으로 사용되기에 여러 위치에서 프로토콜을 타입으로 사용할 수 있다.
> - 함수, 메서드, 이니셜라이저에서 매개변수 타입이나 반환 타입으로 사용될 수 있다.
> - 프로퍼티, 변수, 상수 등의 타입으로 사용될 수 있다.
> - 배열, 딕셔너리 등 컨테이너 요소의 타입으로 사용될 수 있다.

### 가변 메서드 요구
프로토콜이 어떤 타입이든 간에 인스턴스 내부의 값을 변경해야 하는 메서드를 요구하려면 프로토콜의 메서드 정의 앞에 `mutating` 키워드를 명시해야 한다.
참조 타입인 클래스의 메서드 앞에는 `mutating` 키워드를 명시하지 않아도 인스턴스 내부의 값을 바꾸는데 문제 없지만, 값 타입인 구조체, 열거형의 메서드 앞에는 `mutating` 키워드를 붙인 가변 메서드 요구가 필요하다.
프로토콜에 `mutating` 키워드를 사용한 메서드 요구가 있더라도 클래스 구현에서는 `mutating` 키워드를 명시하지 않아도 된다.
~~~swift
// file: "Code05.swift"
protocol Resettable {
    mutating func reset()
}

class Person: Resettable {
    var name: String?
    var age: Int?
    
    func reset() {
        self.name = nil
        self.age = nil
    }
}

struct Point: Resettable {
    var x = 0
    var y = 0
    
    mutating func reset() {
        self.x = 0
        self.y = 0
    }
}
~~~

### 이니셜라이저 요구
프로토콜은 특정 이니셜라이저를 요구할 수 있으며, 이니셜라이저를 정의하지만 구현은 하지 않는다.

#### 구조체의 이니셜라이저 요구 
구조체는 상속할 수 없기 때문에 이니셜라이저 요구에 대한 큰 제약은 없다.
~~~swift
// file: "Code06.swift"
protocol Named {
    var name: String { get }

    init(name: String)
}

struct Pet: Named {
    var name: String

    init(name: String) {
        self.name = name
    }
}
~~~

#### 클래스의 이니셜라이저 요구
~~~swift
// file: "Code07.swift"
class Person: Named {
    var name: String

    required init(name: String) {
        self.name = name
    }
}
~~~

클래스 타입에서 프로토콜의 이니셜라이저 요구에 부합하는 이니셜라이저를 구현할 때는 이니셜라이저의 종류(지정, 편의)는 중요하지 않다. 
그러나 이니셜라이저 요구에 부합하는 이니셜라이저를 구현할 때는 `required` 식별자를 붙인 요구 이니셜라이저로 구현해야 한다.
Code07의 `Person` 클래스를 상속받는 모든 하위 클래스는 `Named` 프로토콜을 준수해야 하며, 이는 곧 상속받는 클래스에 해당 이니셜라이저를 모두 구현해야 한다는 뜻이다. 
만약 클래스 자체가 상속받을 수 없는 `final` 클래스라면 `required` 식별자를 명시할 필요 없다.(상속할 수 없는 클래스의 요구 이니셜라이저 구현은 무의미)
~~~swift
// file: "Code08.swift"
final class Person: Named {
    var name: String

    init(name: String) {
        self.name = name
    }
}
~~~

특정 클래스에 프로토콜이 요구하는 이니셜라이저가 이미 구현되어 있는 상황에서 해당 클래스를 상속받은 클래스가 있다면, `required`와 `overried` 키워드를 모두 명시해 프로토콜에서 요구하는 이니셜라이저를 구현해주어야 한다.
~~~swift
// file: "Code09.swift"
class School {
    var name: String

    init(name: String) {
        self.name = name
    }
}

class MiddleSchool: School, Named {
    required override init(name: String) {
        super.init(name: name)
    }
}
~~~

## 프로토콜의 상속과 클래스 전용 프로토콜
프로토콜은 하나 이상의 프로토콜을 상속받아 기존 프로토콜의 요구사항보다 더 많은 요구사항을 추가할 수 있다.
~~~swift
// file: "Code10.swift"
protocol Readable {
    func read()
}

protocol Writeable {
    func write()
}

protocol ReadSpeakable: Readable {
    func speak()
}

protocol ReadWriteSpeakable: Readable, Writeable {
    func speak()
}

class SomeClass: ReadWriteSpeakable {
    func read() { print("Read") }
    func write() { print("Write") }
    func speak() { print("Speak") }
}
~~~

Code10에서 확인할 수 있듯 모든 프로토콜을 상속받은 `ReadWriteSpeakable` 프로토콜을 채택한 `SomeClass`는 세 프로토콜이 요구하는 모든 메서드를 구현해야 한다.

프로토콜의 상속 리스트에 `class` 키워드를 명시해 프로토콜이 클래스 타입에만 채택될 수 있도록 제한할 수 있다.
클래스 전용 프로토콜로 제한을 주기 위해서는 프로토콜의 상속 리스트 맨 처음에 `class`키워드가 위치해야 한다.
~~~swift
// file: "Code11.swift"
protocol ClassOnlyProtocol: class, Readable, Writeable {
    // Additional requirements...
}

class SomeClass: ClassOnlyProtocol {
    func read() { }
    func write() { }
}

// 컴파일 에러!
struct SomeStruct: ClassOnlyProtocol { }
~~~

## 프로토콜 조합과 프로토콜 준수 확인
하나의 매개변수가 여러 프로토콜을 준수하는 타입이어야 한다면 하나의 매개변수에 여러 프로토콜을 한 번에 조합하여 요구할 수 있다. 
프로토콜을 조합하여 요구할 때 앰퍼샌드(`&`)로 프로토콜을 연결해 명시한다.

또한 특정 클래스의 인스턴스 역할을 할 수 있는지 함께 확인할 수 있다.
~~~swift
// file: "Code12.swift"
protocol Named {
    var name: String { get }
}

protocol Aged {
    var age: Int { get }
}

struct Person: Named, Aged {
    var name: String
    var age: Int
}

class Car: Named {
    var name: String
    
    init(name: String) {
        self.name = name
    }
}

class Truck: Car, Aged {
    var age: Int
    
    init(name: String, age: Int) {
        self.age = age
        super.init(name: name)
    }
}

func celebrateBirthday(to celebrator: Named & Aged) {
    print("Happy birthday \(celebrator.name)!! Now you age \(celebrator.age)")
}

let faker = Person(name: "Lee Sang-hyuk", age: 28)
celebrateBirthday(to: faker)    // Happy birthday Lee Sang-hyuk!! Now you age 28

let myCar = Car(name: "Bentley")
// 컴파일 에러! - Aged 프로토콜 준수 못함
celebrateBirthday(to: myCar)

// 컴파일 에러! - 클래스 & 프로토콜 조합에서 클래스 타입은 한 타입만 조합 가능
var someVariable: Car & Truck & Aged

// Car 클래스의 인스턴스 역할 및 Aged 프로토콜 준수하는 인스턴스만 할당 가능
var someVariable: Car & Aged

someVariable = Truck(name: "Truck", age: 5)
~~~

`is`와 `as` 연산자를 통해 대상이 프로토콜을 준수하는지 확인할 수 있으며, 특정 프로토콜로 캐스팅 할 수 있다.
* `is` 연산자를 통해 해당 인스턴스가 특정 프로토콜을 준수하는지 확인할 수 있다.
* `as?` 다운캐스팅 연산자를 통해 다른 프로토콜로 다운 캐스팅을 시도해볼 수 있다.
* `as!` 다운캐스팅 연산자를 통해 다른 프로토콜로 강제 다운캐스팅을 할 수 있다.
  
~~~swift
// file: "Code13.swift"
print(faker is Named)   // true
print(faker is Aged)    // true

print(myCar is Named)   // true
print(myCar is Aged)    // false

if let castedInstance = faker as? Named { print("\(castedInstance) is Named") }
// Person(name: "Lee Sang-hyuk", age: 28) is Named

if let castedInstance = faker as? Aged { print("\(castedInstance) is Aged") }
// Person(name: "Lee Sang-hyuk", age: 28) is Aged

if let castedInstance = myCar as? Named { print("\(castedInstance) is Named") }
// Car is Named

if let castedInstance = myCar as? Aged { print("\(castedInstance) is Aged") }
// 출력 없음.. 캐스팅 실패
~~~

## 프로토콜의 선택적 요구
프로토콜의 요구사항 중 일부를 선택적으로 요구사항으로 지정할 수 있으며, 선택적 요구사항을 정의하고 싶은 프로토콜은 `objc` 속성이 부여된 프로토콜이어야 한다.

또한 `objc` 속성이 부여되는 프로토콜은 Objective-C 클래스를 상속받은 클래스에서만 채택할 수 있다. 즉, 열거형이나 구조체 등에서는 `objc` 속성이 부여된 프로토콜을 채택할 수 없다.

선택적 요구를 하면 프로토콜을 준수하는 타입에 해당 요구사항을 필수로 구현할 필요가 없다. 선택적 요구사항은 `optional` 식별자를 요구사항 정의 앞에 명시한다.

## 프로토콜 변수와 상수
프로토콜 이름을 타입으로 갖는 변수 또는 상수에는 해당 프로토콜을 준수하는 타입의 어떤 인스턴스라도 할당할 수 있다.
~~~swift
// file: "Code14.swift"
var someNamed: Named = Animal(name: "Animal")
someNamed = Pet(name: "Pet")
someNamed = Person(name: "Person")
someNamed = School(name: "School")
~~~

프로토콜은 프로토콜 이름만으로 인스턴스를 생성하고 초기화할 수 없다. 다만 프로토콜 변수나 상수를 생성하여 특정 프로토콜을 준수하는 타입의 인스턴스를 할당할 수 있다.
---
layout: post
title: "Swift Method"
sitemap: false
image: /assets/img/blog/thumbnail/swift/swift_method.png
categories: [study, swift]
tags: [swift]
description: 본 글에서는 Swift 특정 타입에 관련된 함수 메서드에 대해 다룬다.
---

* toc
{:toc}
## 메서드
메서드는 특정 타입에 관련된 함수를 뜻한다. 클래스, 구조체, 열거형 등은 실행하는 기능을 캡슐화한 **인스턴스 메서드**로 정의할 수 있으며, 타입 자체와 관련된 기능을 실행하는 **타입 메서드**도 정의할 수 있다.

## 인스턴스 메서드
인스턴스 메서드는 특정 타입의 인스턴스에 속한 함수를 뜻한다. 인스턴스 내부의 프로퍼티 값을 변경하거나 특정 연산 결과를 반환하는 등 인스턴스와 관련된 기능을 실행한다. 

인스턴스 메서드는 특정 타입 내부에서 구현되므로 인스턴스가 존재할 때만 호출할 수 있다. 이 점이 함수와 유일한 차이점이다.

### mutating
자신의 프로퍼티 값을 수정할 때 클래스의 인스턴스 메서드는 신경 쓸 필요가 없지만, 구조체나 열거형 등은 값타입이므로 메서드 앞에 `mutating` 키워드를 기입해 해당 메서드가 인스턴스 내부의 값을 변경한다는 것을 명시해야 한다.

~~~swift
// file: "mutating 키워드 사용"
struct LevelStruct {
    var level = 0 {
        didSet { print("Level \(level)") }
    }
    
    mutating func levelUp() {
        print("Level Up!")
        level += 1
    }
    
    mutating func levelDown() {
        print("Level Down")
        level -= 1
        
        if level < 0 { reset() }
    }
    
    mutating func jumpLevel(to: Int) {
        print("Jump to \(to)")
        level = to
    }
    
    mutating func reset() {
        print("Reset!")
        level = 0
    }
}

var levelStructInstance = LevelStruct()
levelStructInstance.levelUp()   // Level Up!
// Level 1
levelStructInstance.levelDown() // Level Down
// Level 0
levelStructInstance.levelDown() // Level Down
// Level -1
// Reset
// Level 0
levelStructInstance.jumpLevel(to: 3)    // Jump to 3
// Level 3
~~~

### self 프로퍼티
모든 인스턴스는 암시적으로 생성된 `self` 프로퍼티를 갖는다. 이는 인스턴스 자기 자신을 가리키는 프로퍼티이며, 인스턴스를 더 명확히 지징하고 싶을 때 사용한다.

만약 메서드 내에 인스턴스 프로퍼티나 매개변수 이름, 메서드 지역변수 이름이 같다면, Swift는 자동으로 메서드 내부에 선언된 지역변수를 먼저 사용하고, 그 다음 메서드 매개변수, 그다음 인스턴스의 프로퍼티를 찾아서 무엇을 지칭하는지 유추한다. 이때 인스턴스의 프로퍼티를 지징하고 싶을 때 `self`프로퍼티를 사용한다.

~~~swift
// file: "self 프로퍼티의 사용"
class LevelClass {
    var level = 0

    func jumpLevel(to level: Int) {
        print("Jump to \(level)")
        self.level = level
    }
}
~~~

`self` 프로퍼티는 값 타입 인스턴스 자체의 값을 치환할 수 있다. 클래스의 인스턴스는 참조 타입이라서 `self` 프로퍼티에 다른 참조를 할당할 수 없지만, 구조체나 열거형 등은 `self` 프로퍼티를 사용하여 자신 자체를 치환할 수도 있다.

~~~swift
// file: "self 프로퍼티와 mutating 키워드"
class LevelClass {
    var level = 0
    
    func reset() { self = LevelClass() }    // 컴파일 에러! - self 프로퍼티 참조 변경 불가
}

struct LevelStruct {
    var level = 0
    
    mutating func levelUp() {
        print("Level Up!")
        level += 1
    }
    
    mutating func reset() {
        print("Rest!")
        self = LevelStruct()
    }
}

var levelStructInstance = LevelStruct()
levelStructInstance.levelUp()       // Level Up!
print(levelStructInstance.level)    // 1

levelStructInstance.reset()         // Reset!
print(levelStructInstance.level)    // 0

enum OnOffSwitch {
    case on, off
    
    mutating func nextState() {
        self = self == .on ? .off : .on
    }
}

var toggle = OnOffSwitch.off
toggle.nextState()
print(toggle)   // on
~~~

### 인스턴스를 함수처럼 호출하는 메서드
**사용자 정의 명목 타입의 호출 가능한 값(Callable values of user-defined nominal type)**을 구현하기 위해 **인스턴스를 함수처럼 호출할 수 있는 메서드(Call-as-function-method)**가 있다.   
특정 타입의 인스턴스를 문법적으로 함수를 사용하는 것처럼 보이게 할 수 있다는 의미이며, 인스턴스를 함수처럼 호출할 수 있도록 하려면 `callAsFunction` 이라는 이름의 메서드를 구현하면 된다. 

해당 메서드는 매개변수와 반환 타입만 다르면 개수에 제한 없이 만들 수 있고, `mutating` 키워드와 `throws`, `rethrows` 기능도 지원한다.

~~~swift
// file: "구조체에 callAsFunction 메서드 구현"
struct Puppy {
    var name = "멍멍이"
    
    func callAsFunction() { print("멍멍") }
    
    func callAsFunction(destination: String) { print("\(destination)(으)로 달려갑니다.") }
    
    func callAsFunction(something: String, times: Int) { print("\(something)(을)를 \(times)번 반복합니다.") }
    
    func callAsFunction(color: String) -> String { "\(color) 응가" }
    
    mutating func callAsFunction(name: String) { self.name = name }
}

var doggy = Puppy()
doggy.callAsFunction()  // 멍멍
doggy()                 // 멍멍

doggy.callAsFunction(destination: "뒷동산")    // 뒷동산(으)로 달려갑니다.
doggy(destination: "집")                      // 집(으)로 달려갑니다.

doggy(something: "재주넘기", times: 3)          // 재주넘기(을)를 3번 반복합니다.
print(doggy(color: "무지개색"))                 // 무지개색 응가
doggy(name: "댕댕이")
print(doggy.name)                             // 댕댕이
~~~

## 타입 메서드
타입 자체에 호출이 가능한 메서드를 **타입 메서드** (흔히 객체지향 프로그래밍에서 지칭하는 클래스 메서드와 유사)라고 한다. 메서드 앞에 `static` 키워드를 사용하여 타입 메서드임을 명시한다.

클래스의 타입 메서드는 `static` 키워드와 `class` 키워드를 사용할 수 있는데 `static` 으로 정의하면 상속 후 메서드 재정의가 불가능하고 `class`로 정의하면 상속 후 메서드 재정의가 가능하다. 

~~~swift
// file: "클래스의 타입 메서드"
class AClass {
    static func staticTypeMethod() { print("AClass staticTypeMethod") }
    
    class func classTypeMethod() { print("AClass classTypeMethod") }
}

class BClass: AClass {
    // 컴파일 에러! - static 타입 메서드 재정의 불가
    override static func staticTypeMethod() { }
    
    override class func classTypeMethod() { print("BClass classTypeMethod") }
}

AClass.staticTypeMethod()   // AClass staticTypeMethod
AClass.classTypeMethod()    // AClass classTypeMethod
BClass.classTypeMethod()    // BClass classTypeMethod
~~~

인스턴스 메서드에서는 `self`가 인스턴스를 가리킨다면 타입 메서드의 `self`는 타입을 지칭한다. 따라서 타입 메서드 내부에서 타입 이름과 `self`는 같은 뜻이므로 타입 메서드에서 `self` 프로퍼티를 사용하면 타입 프로퍼티 및 타입 메서드를 호출할 수 있다.

~~~swift
// file: "타입 프로퍼티와 타입 메서드의 사용"
struct SystemVolume {
    static var volum: Int = 1   // 타입 프로퍼티를 사용하면 언제나 유일한 값이 된다.
    
    static func mute() {
        self.volum = 0
    }
}

class Navigation {
    var volume = 5
    
    func guideWay() { SystemVolume.mute() }
    func finishGuideWay() { SystemVolume.volum = self.volume }
}

SystemVolume.volum = 10

let myNavi = Navigation()
myNavi.guideWay()
print(SystemVolume.volum)   // 0

myNavi.finishGuideWay()
print(SystemVolume.volum)   // 5
~~~
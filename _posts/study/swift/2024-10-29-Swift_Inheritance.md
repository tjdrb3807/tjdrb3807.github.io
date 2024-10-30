---
layout: post
title: "Swift Inheritance"
sitemap: false
image: /assets/img/blog/thumbnail/swift/swift_instance.png
categories: [study, swift]
tags: [swift]
description: 본 글에서는 Swift 사용자 지정 데이터의 인스턴스 생성 및 소멸에 대해 다룬다.
---

* toc
{:toc}

## 🤚 Introduce
클래스는 메서드나 프로퍼티 등을 다른 클래스로부터 **상속**받을 수 있다. 특정 클래스로부터 상속을 받은 클래스를 **하위 클래스(SubClass)**, 하위 클래스에게 특성을 물려준 클래스를 **상위 클래스(SuperClass)**라 표현한다.

Swift의 클래스는 상위 클래스로부터 상속받은 메서드를 호출할 수 있고, 프로퍼티에 접근할 수 있으며 서브스크립트도 사용 가능하다. 또한 상위 클래스로부터 상속받은 메서드, 프로퍼티, 서브스크립트 등을 자신만의 내용으로 **재정의**할 수 있으며, 상속받은 저장, 연산 프로퍼티에 프로퍼티 감시자를 구현할 수도 있다.

다른 클래스로부터 상속 받지 않은 클래스를 **기본 클래스(Base Class)**라 칭하며, 상속을 통해 기본 클래스보다 세분화된 특징과 더 많은 기능을 실행할 수 있는 새로운 하위 클래스를 만들 수 있다.

## ✍🏻 클래스 상속
상속은 기본 클래스를 다른 클래스에서 물려받는 것을 의미한다. 상위 클래스의 메서드, 프로퍼티 등을 재정의하거나, 기본 클래스의 기능이나 프로퍼티를 물려받고 자신의 기능을 추가할 수 있다.

클래스 이름 뒤에 콜론(`:`)을 붙이고 상위 클래스의 이름을 기입하면 뒤에 오는 상위 클래스의 기능을 앞의 하위 클래스가 상속받을 것임을 뜻한다.

~~~swift
// file: "Code01.swift"
class LCKPlayer {
    var name = ""
    var age = 0
    
    var introduction: String { return "이름: \(name), 나이: \(age)" }
    
    func speak() { print("I am LCK player") }
}

class T1Player: LCKPlayer {
    var tier = "Bronze"
    
    func trainig() { print("Trainig hard...") }
}

let faker = LCKPlayer()
faker.name = "Lee Sang-hyuk"
faker.age = 28
print(faker.introduction)   // 이름: Lee Sang-hyuk, 나이: 28
faker.speak()               // I am LCK player

let keria = T1Player()
keria.name = "Ryu Min-seok"
keria.age = 22
keria.tier = "Challenger"
print(keria.introduction)   // 이름: Ryu Min-seok, 나이: 22
keria.speak()               // I am LCK player
keria.trainig()             // Trainig hard...
~~~

위 코드에서 `LCKPlayer` 클래스는 `T1Player` 클래스를 상속받아 상위 클래스가 물려준 프로퍼티와 메서드를 사용할 수 있으며, 자신이 정의한 프로퍼티와 메서드도 사용할 수 있다.

다른 클래스를 상속받으면 똑같은 기능을 구현하기 위해 코드를 다시 작성할 필요가 없으므로 코드의 재사용 및 유지 보수성을 확보할 수 있다.

---
## ✍🏻 재정의
하위 클래스는 상위 클래스로부터 상속받은 특성(인스턴스 메서드, 타입 메서드, 인스턴스 프로퍼티, 타입 프로퍼티, 서브스크립트 등)을 그대로 사용하지 않고 자신만의 기능으로 **재정의(Override)**하여 사용할 수 있다.

상속받은 특성들을 재정의하기 위해서는 새로운 정의 앞에 `override` 키워드를 사용한다. `override` 키워드는 Swift 컴파일러가 상위 클래스에 해당 특성이 있는지 확인 후 재정의하게 된다. 이때 상위 클래스에 재정의할 특성이 없는경우 컴파일 에러가 발생한다.

만약 하위 클래스에서 재정의할 때 상위 클래스의 특성을 사용하고 싶을 경우 `super` 프로퍼티를 사용한다. 이때 `super` 키워드를 타입 메서드 내에서 사용하면 상위 클래스의 타입 메서드와 타입 프로퍼티에 접근할 수 있으며, 인스턴스 메서드 내에서 사용하면 상위 클래스의 인스턴스 메서드와 인스턴스 프로퍼티, 서브스크립트에 접근할 수 있다.

### 메서드 재정의
상위 클래스로부터 상속받은 인스턴스 메서드나 타입 메서드를 하위 클래스에서 기획 의도에 맞도록 재정의 할 수 있다.

```swift
// file: Code02.swift
class LCKPlayer {
    var name = ""
    var age = 0
    
    var introduction: String { return "name: \(name), age: \(age)" }
    
    func speak() { print("I am LCK player") }
    
    class func introduceClass() -> String { "It's LCK player class." }
}

class T1Player: LCKPlayer {
    var tier = "Bronze"
    
    func trainig() { print("Trainig hard...") }
    
    override func speak() { print("I am T1 Player.") }
}

class WorldPlayer: T1Player {
    var seed = ""
    
    class func introduceClass() { print(super.introduceClass()) }
    
    override class func introduceClass() -> String { "It's World player class." }
    
    override func speak() {
        super.speak()
        print("I am World player.")
    }
}

let showMaker = LCKPlayer()
showMaker.speak()   // I am LCK player

let owner = T1Player()
owner.speak()       // I am T1 Player.

let faker = WorldPlayer()
faker.speak()       // I am T1 Player.
                    // I am World player.
print(LCKPlayer.introduceClass())              // It's LCK player class.
print(T1Player.introduceClass())               // It's LCK player class.
print(WorldPlayer.introduceClass() as String)  // It's World player class.
WorldPlayer.introduceClass() as Void           // It's LCK player class.


```

위 코드를 확인해보면 `T1Player` 클래스에서 `LCKPlayer` 클래스에 정의된 `speak()` 메서드를 재정의 했으며, `WorldPlayer` 클래스에서는 `LCKPlayer` 클래스의 `introduceClass()` 메서드를 재정의했다.

`T1Player` 클래스에서 재정의한 `speak()` 메서드는 `WorldPlayer` 클래스에 상속되었으므로 `WorldPlayer` 클래스의 인스턴스가 `speak()` 메서드를 호출하면 `T1Player` 클래스에서 재정의한 메서드가 호출된다.

상위 클래스의 메서드에 접근하기 위해서는 `WorldPlayer` 클래스의 `speak()` 인스턴스 메서드와 `introduceClass()` 타입 메서드처럼 `super` 프로퍼티를 사용하면 된다.

### 프로퍼티 재정의
프로퍼티를 재정의 한다는 것은 프로퍼티의 **접근자(Getter)**, **설정자(Setter)**, **프로퍼티 감시자(Property Observer)**등을 재정의하는 것을 의미하며, 상위 클래스로부터 상속받은 인스턴스 프로퍼티나 타입 프로퍼티를 하위 클래스에서 기획 의도에 맞게 재정의할 수 있다.

프로퍼티를 상속받은 하위 클래스는 상위 클래스의 프로퍼티 종류(저장, 연산 등)는 알지 못하고 프로퍼티 이름과 타입만 알기 때문에 상위 클래스에서 정의한 저장, 연산 프로퍼티의 접근자와 설정자를 재정의 할 수 있다. 이때 재정의하려는 프로퍼티는 상위 클래스의 프로퍼티 이름과 타입이 일치해야 하며, 상위 클래스에 없는 프로퍼티를 재정의하려고 하면 컴파일 에러가 발생한다.

상위 클래스에 위치한 프로퍼티가 읽기 전용이라도 하위 클래스에서 읽고 쓰기가 가능한 프로퍼티로 재정의 할 수 있다. 하지만, 읽기 쓰기 모두 가능했던 상위 클래스의 프로퍼티를 하위 클래스에서 읽기 전용으로 재정의해줄 수는 없다.

읽기 쓰기가 모두 가능한 프로퍼티를 재정의할 때 설정자만 별도로 재정의할 수는 없고 접근자와 설정자 모두 재정의해야 한다. 접근자에 별도의 변경 기능이 필요 없다면 `super.someProperty`와 같은 상위 클래스 접근자를 사용하여 값을 받아와 반환해준다.

~~~swift
// file: "Code03.swift"
class Player {
    var name: String = ""
    var age: Int = 0
    var koreanAge: Int { self.age + 1 } // (읽기 전용) 연산 프로퍼티
    
    var introduction: String { "Name: \(name), Age: \(age)" }
}

class LCKPlayer: Player {
    var pogPoint: Int = 1000
    
    override var introduction: String { super.introduction + " " + "POG Point: \(self.pogPoint)" }
    
    // 재정의 - (읽기, 쓰기) 연산 프로퍼티 
    // 접근자, 설정자 모두 재정의
    override var koreanAge: Int {
        get { super.koreanAge }
        set { self.age = newValue - 1 }
    }
}

let owner = Player()
owner.name = "Moon Hyun-joon"
owner.age = 21
owner.koreanAge = 22        // 컴파일 에러! - (읽기 전용) 연산 프로퍼티
print(owner.introduction)   // Name: Moon Hyun-joon, Age: 21
print(owner.koreanAge)      // 22

let faker = LCKPlayer()
faker.name = "Lee Sang-hyuk"
faker.age = 28
faker.koreanAge = 27
print(faker.introduction)   // Name: Lee Sang-hyuk, Age: 26 POG Point: 1000
print(faker.koreanAge)      // 27
~~~

### 프로퍼티 감시자 재정의
프로퍼티 감시자를 하위 클래스에서 재정의할 때 상위 클래스에 정의된 프로퍼티의 종류(저장, 연산)는 상관없다. 하지만, 상수 저장 프로퍼티나 읽기 전용 연산 프로퍼티는 값을 설정할 수 없으므로 `willSet`이나 `didSet` 메서드를 사용한 프로퍼티 감시자를 사용할 수 없다. 따라서 상수 저장 프로퍼티 또는 읽기 전용 연산 프로퍼티는 프로퍼티 감시자를 재정의할 수 없다.

프로퍼티 감시자를 재정의해도 상위 클래스에 정의한 프로퍼티 감시자도 호출된다.

프로퍼티 접근자와 프로퍼티 감시자는 동시에 재정의할 수 없다. 둘 다 호출되길 원하면 재정의하는 접근자에 프로퍼티 감시자 역할을 구현해야 한다.

~~~swift
// file: "Code04.swift"
class Player {
    var name: String = ""
    // (저장) 프로퍼티 - 프로퍼티 감시자
    var age: Int = 0 {
        didSet { print("Player age: \(self.age)") }
    }
    // (읽기 전용) 연산 프로퍼티
    var koreanAge: Int { self.age + 1 }
    // (읽기, 쓰기) 연산 프로퍼티
    var fullNmae: String {
        get { self.name }
        set { self.name = newValue }
    }
}

class LCKPlayer: Player {
    var pogPoint: Int = 1000
    
    // (저장) 프로퍼티 - 프로퍼티 감시자 재정의
    override var age: Int {
        didSet { print("LCKPlayer age: \(self.age)") }
    }
    
    // (읽기 전용) 연산 프로퍼티 -> (읽기, 쓰기) 연산 프로퍼티 재정의
    override var koreanAge: Int {
        get { super.koreanAge }
        set { self.age = newValue - 1 }
    }
    
    override var fullNmae: String {
        didSet { print("Full Name: \(self.fullNmae)") }
    }
}

let owner = Player()
owner.name = "Hyun-joon"
owner.age = 21                      // Player age: 21
owner.fullNmae = "Moon Hyun-joon"
print(owner.koreanAge)              // 22

let faker = LCKPlayer()
faker.name = "Sang-hyuk"
faker.age = 28                      // Player age: 28
                                    // LCKPlayer age: 28
faker.koreanAge = 29                // Player age: 28
                                    // LCKPlayer age: 28
faker.fullNmae = "Lee Sang-hyuk"    // Full Name: Lee Sang-hyuk
print(faker.koreanAge)              // 29
~~~

위 코드에서 `LCKPlayer` 인스턴스의 `age` 프로퍼티에 값을 할당할 때, 상위 클래스(`Player`) 에서 정의한 프로퍼티 감시자와 하위 클래스(`LCKPlayer`)에서 재정의한 프로퍼티 감시자 둘 다 동작하는 것을 확인할 수 있다.

또한 상위 클래스에서 연산 프로퍼티로 정의했던 `fullName` 프로퍼티를 상속받아 하위 클래스에서 프로퍼티 감시자를 재정의했음을 확인할 수 있다.

### 서브스크립트 재정의
매개변수와 반환 타입이 다르면 다른 서브스크립트로 취급하므로, 하위 클래스에서 재정의하려는 서브 스크립트라면 상위 클래스에 정의된 서브스크립트의 매개변수와 반환 타입이 같아야 한다.

### 재정의 방지
상위 클래스를 상속받는 하위 클래스에서 특정 특성을 재정의할 수 없도록 제한하려면 재정의를 방지하고 싶은 특성 앞에 `final` 키워드를 명시하면 되며, 재정의를 방지한 특성을 하위 클래스에서 재정의하려 하면 컴파일 에러가 발생한다.

클래스 상속 및 재정의할 수 없도록 설정하려면 `class` 키워드 앞에 `final` 키워드를 명시하면 되며, 해당 클래스는 하위 클래스를 가질 수 없으며, 해당 클래스를 다른 클래스가 상속받으려 하면 컴파일 에러가 발생한다.

~~~swift
// file: "Code06.swift"
class T1Player {
    final var name: String = ""
    final func speak() { print("I am T1 Player!") }
}

final class LCKPlayer: T1Player {
    // 컴파일 에러! - 상위 클래스의 name 프로퍼티 재정의 방지 설정
    override var name: String {
        // ...
    }
}

// 컴파일 에러! - T1Player 클래스 재정의 방지 설정
class WorldPlayer: LCKPlayer { /... }
~~~

---
## 클래스의 이니셜라이저

값 타입의 이니셜라이저는 이니셜라이저 위임을 위해 이니셜라이저끼리 구분할 필요가 없지만 클래스(참조 타입)에서는 **지정 이니셜라이저**와 **편의 이니셜라이저**로 역할을 구분한다. 또한, 값 타입의 이니셜라이저는 상속을 고려할 필요가 없지만 클래스는 상속이 가능하므로 상속을 받았을 때 이니셜라이저를 어떻게 재정의하는지가 관건이다.

### 지정 이니셜라이저
**지정 이니셜라이저(Designated Initialiazer)**는 해당 이니셜라이저가 정의된 클래스의 모든 프로퍼티를 초기화해야 하는 의무가 있으며, 필요에 따라 상위 클래스의 이니셜라이저를 호출해 상위 클래스 체인까지 초기화 작업을 할 수 있다.

모든 클래스는 하나 이상의 지정 이니셜라이저를 갖으며, 상위 클래스의 지정 이니셜라이저가 하위 클래스의 지정 이니셜라이저 역할을 수행할 수 있다면, 하위 클래스는 지정 이니셜라이저를 갖지 않을 수 있다. (위 상황은 상위 클래스로부터 상속받은 프로퍼티를 제외하고 옵셔널 저장 프로퍼티 외에 다른 저장 프로퍼티가 없을 경우이다.)

~~~swift
// file: "지정 이니셜라이저 정의.swift"
init(Parameter) {
    // 초기화 구문
}
~~~

### 편의 이니셜라이저
**펀의 이니셜라이저(Convenience Initializer)**는 클래스 초기화 지원 보조 역할을 한다.

지정 이니셜라이저의 매개변수를 기본값으로 설정하여 편의 이니셜라이저와 동일한 클래스에서 지정 이니셜라이저를 호출하도록 편의 이니셜라이저를 정의할 수 있다.

지정 이니셜라이저의 매개변수가 많아 외부에서 일일이 전달인자를 전달하기 어렵거나 특정 목적에서 사용하기 위해 편의 이니셜라이저를 설계할 수도 있다.

지정 이니셜라이저를 사용하면 인스턴스를 생성할 때마다 전달인자로 초기값을 전달해야 하지만 편의 이니셜라이저를 사용하면 항상 가은 값으로 초기화가 가능하다.

편의 이니셜라이저는 클래스를 구현하는데 있어 필수 요소는 아니다. 설계자의 의도대로 외부에서 사용하길 원하거나 인스턴스 생성 코드를 작성하는 수고를 덜 때 유용하게 사용할 수 있다.

편의 이니셜라이저는 `convenience` 지정자를 `init` 키워드 앞에 명시하면 된다.

~~~swift
// file: "편의 이니셜라이저 정의.swift"
convenience init(Parameter) {
    // 초기화 구문
}
~~~

### 클래스의 초기화 위임
지정 이니셜라이저와 편의 이니셜라이저 사이의 관계를 단순화 하기위해 Swift는 초기화 사이의 위임 호출에 대한 3가지 규착을 적용한다.
1. 하위 클래스의 지정 이니셜라이저는 상위 클래스의 지정 이니셜라이저를 반드시 호출해야 한다.
2. 편의 이니셜라이저는 자신을 정의한 클래스의 다른 이니셜라이저를 반드시 호출해야 한다.
3. 편의 이니셜라이저는 궁극적으로 지정 이니셜라이저를 반드시 호출해야 한다.

위 세 가지 규칙을 다음처럼 생각해볼 수도 있다.
* 지정 이니셜라이저 또는 편의 이니셜라이저는 지정 이니셜라이저에게 초기화를 반드시 위임한다.
* 편의 이니셜라이저는 초기화를 반드시 지정 이니셜라이저 또는 편의 이니셜라이저에 위임한다.


### 2단계 초기화
### 이니셜라이저 상속 및 재정의
### 이니셜라이저 자동 상속
### 요구 이니셜라이저
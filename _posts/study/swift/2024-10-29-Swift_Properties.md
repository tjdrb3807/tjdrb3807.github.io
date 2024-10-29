---
layout: post
title: "Swift Properties"
sitemap: false
image: /assets/img/blog/thumbnail/swift/swift_properties.png
categories: [study, swift]
tags: [swift]
description: 본 글에서는 사용자 지정 데이터에 관련된 프로퍼티의 다양한 종류에 대해 다룬다.
---

* toc
{:toc}

## 저장 프로퍼티
클래스 또는 구조체의 인스턴스와 연관된 값을 저장하는 단순한 개념의 프로퍼티.      

* 변수 저장 프로퍼티: `var` 키워드 사용
* 상수 저장 프로퍼티: `let` 키워드 사용

저장 프로퍼티를 정의할 때 프로퍼티 **기본값**과 **초기값**을 지정할 수 있다.   

### 저장 프로퍼티 선언
~~~swift
// file: "저장 프로퍼티의 선언 및 인스턴스 생성"
struct CoordinatePoint {
    var x: Int  // 변수 저장 프로퍼티
    var y: Int  // 변수 저장 프로퍼티
}

// 구조체에는 기본적으로 저장 프로퍼티를 매개변수로 갖는 이니셜라이저가 존재
let fakerPoint = CoordinatePoint(x: 10, y: 5)

class Position {
    var point: CoordinatePoint  // 변수 저장 프로퍼티
    let name: String            // 상수 저장 프로퍼티
    
    // 프로퍼티 기본값을 지정해주지 않는다면 이니셜라이저를 별도로 생성해야한다.
    init(point: CoordinatePoint, name: String) {
        self.point = point
        self.name = name
    }
}

let fakerPosition = Position(point: fakerPoint, name: "faker")
~~~

### 저장 프로퍼티 초기값 할당

구조체는 구조체 내에 선언한 프로퍼티에 맞는 **멤버와이즈 이니셜라이저(Memberwies Initializer)**를 제공하지만 클래스는 저장 프로퍼티의 초기값을 지정해 별도의 이니셜라이저 구현을 생략할 수 있다.

~~~swift
// file: "저장 프로퍼티의 초기값 지정"
struct CoordinatePoint {
    var x: Int = 0  // 변수 저장 프로퍼티 - 초기값 할당
    var y: Int = 0  // 변수 저장 프로퍼티 - 초기값 할당
}

// 프로퍼티의 초기값을 할당했다면 굳이 전달인자로 초기값을 넘길 필요 없다.
let fakerPoint = CoordinatePoint()

// 기존 초기값을 할당할 수 있는 멤버와이즈 이니셜라이저 사용 가능
let kerialPoint = CoordinatePoint(x: 10, y: 5)

print("faker's point: (\(fakerPoint.x), \(fakerPoint.y))")      // faker's point: (0, 0)
print("keria's point: (\(kerialPoint.x), \(kerialPoint.y))")    // keria's point: (10, 5)

class Position {
    var point: CoordinatePoint = CoordinatePoint()  // 변수 저장 프로퍼티 - 초기값 할당
    var name: String = "Unknown"
}

// 클래스 인스턴스 프로퍼티에 초기값을 지정했으므로 사용자 정의 이니셜라이저를 사용하지 않아도 된다.
let ownerPosition = Position()
ownerPosition.point = kerialPoint
ownerPosition.name = "Owner"
~~~

### 옵셔널 저장 프로퍼티

인스턴스를 생성할 때 이니셜라이저를 통해 초기값을 전달하는 이유는 프로퍼티가 옵셔널이 아닌 값으로 선언되어 있기 때문이다. 따라서 인스턴스가 생성되는 시점에 프로퍼티의 값이 반드시 있어야 한다. 

만약 저장 프로퍼티의 값이 옵셔널이라면 굳이 초기값을 넣어주지 않아도 된다. 즉, 이니셜라이저에서 옵셔널 프로퍼티에 값을 할당해주지 않아도 된다.

~~~swift
// file: "옵셔널 저장 프로퍼티"
// 좌표
struct CoordinatePoint {
    // 위치는 x, y 값이 모두 있어야 하므로 옵셔널이면 안 된다.
    var x: Int
    var y: Int
}

// 사람의 위치 정보
class Position {
    var point: CoordinatePoint? // 현재 사람의 위치는 모를 수도 있다. - 옵셔널
    let name: String
    
    init(name: String) {
        self.name = name
    }
}

// 이름은 필수지만 위치는 모를 수 있다.
let fakerPosition = Position(name: "Lee Sang-hyuk")

// 위치를 알게되면 그 때 위치 값을 할당한다.
fakerPosition.point = CoordinatePoint(x: 20, y: 10)
~~~

옵셔널과 이니셜라이저를 적절히 사용하면 다를 프로그래머가 사용할 때, 작성 의도대로 구조체와 클래스를 사용할 수 있도록 유도할 수 있다.

## 지연 저장 프로퍼티
**지연 저장 프로퍼티(Lazy Stored Properties)**는 필요할 때 값이 할당되는 기능을 제공한다.   
지연 저장 프로퍼티는 호출이 있어야 값을 초기화하며, `lazy` 키워드를 사용한다.

상수는 인스턴스가 생성되기 전에 초기화해야 하므로 필요할 때 값을 할당하는 지연 저장 프로퍼티와는 맞지 않으며, `var` 키워드를 사용하여 변수로 정의한다.

지연 저장 프로퍼티는 클래스 인스턴스의 저장 프로퍼티로 다른 클래스 인스턴스나 구조체 인스턴스를 할당해야 할 때 주로 사용된다. 인스턴스를 초기화하면서 저장 프로퍼티로 쓰이는 인스턴스들이 한 번에 생성되어야 하거나 모든 저장 프로퍼티를 사용할 필요가 없을 때 지연 저장 프로퍼티를 사용한다.    
지연 저장 프로퍼티를 적절하게 사용하면 불필요한 성능저하나 공간 낭비를 줄일 수 있다.
~~~swift
// file: "지연 저장 프로퍼티"
class Position {
    lazy var point: CoordinatePoint = CoordinatePoint() // 지연 저장 프로퍼티
    let name: String

    init(name: String) {
        self.name = name
    }
}

let fakerPoint = Position(name: "Lee Sang-hyuk")

print(fakerPoint.point) // 해당 시점에 point 프로퍼티로 처음 접근하며 point 지연 저장 프로퍼티의 CoordinatePoint가 생성된다.
~~~

> **NOTE_다중 스레드와 지연 저장 프로퍼티**
>
> 다중 스레드 환경에서 지연 저장 프로퍼티에 동시에 접근할 때는 한 번만 초기화된다는 보장이 없다. 생성되지 않은 지연 저장 프로퍼티에 많은 스레드가 비슷한 시점에 접근한다면 여러 번 초기화될 수 있다.

## 연산 프로퍼티
연산 프로퍼티는 특정 상태에 따른 값을 연산하는 프로퍼티이다.    
인스턴스 내/외부의 값을 연산하여 적절한 값을 돌려주는 **접근자(Getter)**의 역할이나 은닉화된 내부의 프로퍼티 값을 간접적으로 설정하는 **설정자(Setter)**의 역할을 수행한다.

### 메서드 vs 연산 프로퍼티
인스턴스 외부에서 메서드를 통해 인스턴스 내부의 값에 접근하려면 메서드를 두 개(접근자, 설정자) 구현해야 한다. 또한 두 개의 메서드를 구현했더라도 두 메서드가 분산 구현되어 코드의 가독성이 떨어질 수 있다. 다른 사람의 코드를 보는 입장에서 연산 프로퍼티가 메서드 형식보다 직관적이다.

다만 연산 프로퍼티는 접근자인 `get`메서드만 구현해 읽기 전용 상태로 설정하는 것은 쉽지만, 쓰기 전용 상태로 구현할 수 없다는 단점이 있다.

~~~swift
// file: "메서드로 구현된 접근자와 설정자"
struct CoordinatePoint {
    var x: Int  // (변수) 저장 프로퍼티
    var y: Int  // (변수) 저장 프로퍼티
    
    func oppositePoint() -> Self { CoordinatePoint(x: -x, y: -y) }
    
    mutating func setOppositePoint(_ opposite: CoordinatePoint) {
        x = -opposite.x
        y = -opposite.y
    }
}

var fakerPoint = CoordinatePoint(x: 10, y: 20)
print("current Point: (\(fakerPoint.x). \(fakerPoint.y))")  // current Point: (10. 20)
print("opposite Point: (\(fakerPoint.oppositePoint().x), \(fakerPoint.oppositePoint().y))") // opposite Point: (-10, -20)

fakerPoint.setOppositePoint(CoordinatePoint(x: 15, y: 10))
print("current Point: (\(fakerPoint.x), \(fakerPoint.y))")  // current Point: (-15, -10)
~~~

~~~swift
// file: "연산 프로퍼티의 정의와 사용"
struct CoordinatePoint {
    var x: Int  // (변수) 저장 프로퍼티
    var y: Int  // (변수) 저장 프로퍼티
    var oppositPoint: Self {                    // 연산 프로퍼티
        get { CoordinatePoint(x: -x, y: -y) }   // 접근자
        set(opposit) {                          // 설정자
            x = -opposit.x
            y = -opposit.y
        }
    }
}

var fakerPoint = CoordinatePoint(x: 10, y: 20)
print("current Point: (\(fakerPoint.x). \(fakerPoint.y))")  // current Point: (10. 20)
print("opposite Point: (\(fakerPoint.oppositPoint.x), \(fakerPoint.oppositPoint.y))")   // opposite Point: (-10, -20)

fakerPoint.oppositPoint = CoordinatePoint(x: 15, y: 10)
print("current Point: (\(fakerPoint.x). \(fakerPoint.y))")  // current Point: (-15. -10)
~~~

연산 프로퍼티를 사용하면 위 코드처럼 하나의 프로퍼티에 접근자와 설정자 모두 모여있고, 해당 프로퍼티가 어떤 역할을 하는지 직관적으로 표현이 가능하다. 또한 인스턴스를 사용하는 입장에서도 저장 프로퍼티처럼 편하게 접근할 수 있다.

### 설정자의 매개변수
연산 프로퍼티 설정자의 매개변수로 원하는 이름을 소괄호 안에 명시하면 `set` 메서드 내부에서 전달인자로 사용할 수 있다.    
매개변수 이름을 별도로 지정하지 않으면 관용적으로 `newValue`로 매개변수 이름을 대신할다.

~~~swift
// file: "매개변수 이름을 생략한 설정자"
struct CoordinatePoint {
    var x: Int  // (변수) 저장 프로퍼티
    var y: Int  // (변수) 저장 프로퍼티
    var oppositPoint: Self {                    // 연산 프로퍼티
        get { CoordinatePoint(x: -x, y: -y) }   // 접근자
        set {                                   // 설정자
            x = -newValue.x
            y = -newValue.y
        }
    }
}
~~~

### 읽기 전용 연산 프로퍼티
연산 프로퍼티를 읽기 전용으로 구현하려면 `get` 메서드만 정의한다.

~~~swift
// file: "읽기 전용 연산 프로퍼티"
struct CoordinatePoint {
    var x: Int  // (변수) 저장 프로퍼티
    var y: Int  // (변수) 저장 프로퍼티
    var oppositPoint: Self {                    // (읽기 전용) 연산 프로퍼티
        get { CoordinatePoint(x: -x, y: -y) }   // 접근자
    }
}

var fakerPoint = CoordinatePoint(x: 10, y: 20)
print("current Point: (\(fakerPoint.x). \(fakerPoint.y))")  // current Point: (10. 20)
print("opposite Point: (\(fakerPoint.oppositPoint.x), \(fakerPoint.oppositPoint.y))")   // opposite Point: (-10, -20)

// 컴파일 에러! - 설정자를 구현하지 않음
fakerPoint.oppositPoint = CoordinatePoint(x: 15, y: 10)
~~~

## 프로퍼티 감시자
**프로퍼티 감시자(Property Observers)**를 사용하면 프로퍼티의 값이 변경됨에 따라 구현한 작업을 취할 수 있다. 프로퍼티 감시자는 프로퍼티의 값이 새로 할당될 때마다 호출하며, 변경되는 값이 현재의 값과 같더라도 호출된다.

프로퍼티 감시자는 저장 프로퍼티, 재정의해 상속받은 저장 프로퍼티 또는 연산 프로퍼티에 적용할 수 있으며, 상속받지 않은 연산 프로퍼티는 접근자와 설정자를 통해 프로퍼티 감시자를 구현할 수 있으므로 프로퍼티 감시자기능을 제공하지 않는다.

### willSet and didSet
프로퍼티 감시자에는 프로퍼티의 값이 변경되기 직전에 호출하는 `willSet` 메서드와 프로퍼티의 값이 변경된 직후에 호출되는 `didSet` 메서드가 존재한다.

`willSet`메서드와 `didSet`메서드에는 매개변수가 하나씩 존재하며, `willSet`메서드에 전달되는 전달인자는 프로퍼티가 **변경될 값**이고, `didSet`메서드에 전달되는 전달인자는 프로퍼티가 **변경되기 전의 값**이다.    
만약 메서드에 매개변수 이름을 별도로 지정하지 않으면 `willSet`메서드에는 `newValue`, `didSet`메서드에는 `oldValue` 매개변수 이름이 자동으로 지정된다.

> **NOTE_oldValue와 didSet**
> 
> `didSet` 감시자 코드 블록 내부에서 `oldValue` 값을 참조하지 않거나 매개변수 목록에 명시적으로 매개변수를 적지 않으면 `didSet` 코드 블록이 실행되지 않는다.

~~~swift
// file: "프로퍼티 감시자"
class Account {
    var credit = 0 {    // (변수) 저장 프로퍼티 - 프로퍼티 감시자
        willSet { print("잔액이 \(credit)원에서 \(newValue)원으로 변경될 예정입니다.") }
        didSet { print("잔액이 \(oldValue)원에서 \(credit)원으로 변경되었습니다.") }
    }
}

let myAccount = Account()

// 잔액이 0원에서 1000원으로 변경될 예정입니다.
myAccount.credit = 1000
// 잔액이 0원에서 1000원으로 변경되었습니다.
~~~

### 상속받은 연산 프로퍼티의 프로퍼티 감시자
클래스를 상속받았다면 기존 연산 프로퍼티를 재정의하여 프로퍼티 감시자를 구현할 수 있으며, 이때 연산 프로퍼티를 재정의해도 기존의 연산 프로퍼티 기능(`get`, `set`)은 동작한다.

~~~swift
// file: "상속받은 연산 프로퍼티 - 프로퍼티 감시자"
class Account {
    var credit = 0 {    // (번수) 저장 프로퍼티 - 프로퍼티 감시자
        willSet { print("잔액이 \(credit)원에서 \(newValue)원으로 변경될 예정입니다.") }
        didSet { print("잔액이 \(oldValue)원에서 \(credit)원으로 변경되었습니다.") }
    }
    
    var dollorValue: Double {   // (상속받지 않은) 연산 프로퍼티
        get { Double(credit) / 1000.0 }
        set {
            credit = Int(newValue * 1000.0)
            print("잔액을 \(newValue)달러로 변경 중입니다.")
        }
    }
}

class ForeignAccount: Account { // (상속받은) 연산 프로퍼티 - 프로퍼티 감시자
    override var dollorValue: Double {
        willSet { print("잔액이 \(dollorValue)달러에서 \(newValue)달러로 변경될 예정입니다.") }
        didSet { print("잔액이 \(oldValue)달러에서 \(dollorValue)달러로 변경되었습니다.") }
    }
}

let myAccount = ForeignAccount()

// 잔액이 0원에서 1000원으로 변경될 예정입니다.
myAccount.credit = 1000
// 잔액이 0원에서 1000원으로 변경되었습니다.

//잔액이 1.0달러에서 2.0달러로 변경될 예정입니다.
//잔액이 1000원에서 2000원으로 변경될 예정입니다.
//잔액이 1000원에서 2000원으로 변경되었습니다.
myAccount.dollorValue = 2   // 잔액을 2.0달러로 변경 중입니다.
// 잔액이 1.0달러에서 2.0달러로 변경되었습니다.
~~~

> **NOTE_입출력 매개변수와 프로퍼티 감시자**
>
> 프로퍼티 감시자가 있는 프로퍼티를 함수의 입출력 매개변수의 전달인자로 전달하면 항상 `willSet`과 `didSet` 메서드를 호출한다. 이는 함수 내부에서 전달인자 값의 변경 유무와 상관없이 함수가 종료되는 시점에 값을 다시 쓰기 때문이다.

## 전역변수와 지역변수
프로퍼티와 프로퍼티 감시자는 전역변수와 지역변수 모두에 사용할 수 있다. 함수나 메서드, 클로저, 클래스, 구조체, 열거형 등의 범위 안에 포함되지 않았던 변수나 상수는 모두 전역변수 또는 전역상수에 해당된다.

또한 전역 변수 또는 지역 변수는 **저장 변수**라고 한다. 저장 변수는 저장 프로퍼티처럼 값을 저장하는 역할하며, 전역변수나 지역변수를 연산변수로 구현할 수도 있고, 프로퍼티 감시자를 구현할 수도 있다.

> **NOTE_전역변수, 상수의 지연 저장**
>
> 전역변수 또는 전역상수는 지연 저장 프로퍼티처럼 처음 접근할 때 최초로 연산이 이루어진다. 
> 따라서 `lazy`키워드를 사용하여 연산을 늦출 필요가 없다.
> 반대로 지역변수 및 지역상수는 지연 연산이 되지 않는다.
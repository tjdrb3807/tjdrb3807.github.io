---
layout: post
title: "Swift Instance"
sitemap: false
image: /assets/img/blog/thumbnail/etc/git_section07.png
categories: [study, swift]
tags: [swift]
description: 본 글에서는 Git이 각각의 Branch를 합치는 두 가지 방법에 대해 다룬다.
---

* toc
{:toc}

## Intor
**초기화(Initialiaztion)**는 클래스나 구조체, 열거형의 인스턴스를 사용하기 위한 준비 과정이며, 초기화가 완료된 인스턴스는 사용 후 소멸 시점이 오면 사라진다. 본 글에서는 인스턴스를 생성하는 방법과 클래스의 인스턴스가 소멸할 떄 어떤 프로세스가 진행되는지 확인해본다.

## 인스턴스 생성
초기화 과정은 새로운 인스턴스를 사용할 준비를 하기 위해 저장 프로퍼티의 초기값 설정 등의 작업을 한다. **이니셜라이저(Initializer)**를 정의하면 초기화 과정을 직접 구현할 수 있으며, 구현된 이니셜라이저는 새로운 인스턴스를 생성할 수 있는 메서드가 된다.   
Swift의 이니셜라이저의 역할은 인스턴스의 첫 사용을 초기화 하는 것 뿐이므로 별도의 반환 값이 존재하지 않는다.

이니셜라이저는 메서드지만 `func` 키워드가 아닌 `init` 키워드를 사용해 이니셜라이저 메서드임을 표시하며, 클래스, 구조체, 열거형 등의 구현부 또는 해당 타입의 익스텐션 구현부에 위치한다. 
> Tip. 클래스의 지정 이니셜라이저는 익스텐션에서 구현할 수 없다.

~~~swift
// file: "클래스, 구조체, 열거형의 기본적인 형태의 이니셜라이저"
class SomeClass {
    init() {
        // 초기화할 때 필요한 코드
    }
}

struct SomeStruct {
    init() {
        // 초기화할 때 필요한 코드
    }
}

enum SomeEnum {
    case someCase

    init() {
        self = .someCase    // 열거형은 초기화할 때 반드시 case중 하나가 되어야 한다.
        // 초기화할 때 필요한 코드
    }
}
~~~

### 프로퍼티 기본값
구조체와 클래스의 인스턴스는 처음 생성할 때 옵셔널 저장 프로퍼티를 제외한 모든 저장 프로퍼티에 **초기값(Initial Value)**이 할당되야 하며, 초기화 이후 값이 확정되지 않은 저장 프로퍼티는 존재할 수 없다.

프로퍼티를 정의할 때 **프로퍼티 기본값(Default Value)**을 할당하면 이니셜라이저에서 별도로 초기값을 지정하지 않아도 프로퍼티 기본값으로 저장 프로퍼티의 값이 초기화된다.

> **NOTE_초기화와 프로퍼티 감시자**
>
> 이니셜라이저를 통해 초기값을 할당하거나, 프로퍼티 기본값을 통해 처음의 저장 프로퍼티가 초기화될 때는 프로퍼티 감시자 메서드가 호출되지 않는다.

### 이니셜라이저 매개변수
매개변수를 전달받아 인스턴스를 초기화하는 과정에 필요한 값을 전달받을 수 있다. 

~~~swift
// file: "이니셜라이저 메개변수"
struct Area {
    var squareMeter: Double
    
    init(fromPy py: Double) { squareMeter = py * 3.3058 }  // 첫 번째 이니셜라이저
    
    init(fromSquareMeter squareMeter: Double) { self.squareMeter = squareMeter }    // 두 번째 이니셜라이저
    
    init(value: Double) { squareMeter = value } // 세 번째 이니셜라이저
    
    init(_ value: Double) { squareMeter = value }   // 네 번째 이니셜라이저
}

let roomOne = Area(fromPy: 15.0)
print(roomOne.squareMeter)  // 49.587

let roomTwo = Area(fromSquareMeter: 33.06)
print(roomOne.squareMeter)  // 49.587

let roomThree = Area(value: 30.0)
let roomFour = Area(55.0)

Area()  // 컴파일 에러!
~~~

위 코드처럼 **사용자 정의 이니셜라이저**를 만들면 기존의 기본 이니셜라이저(`inint()`)는 별도로 구현하지 않는 이상 사용할 수 없다.

### 옵셔널 프로퍼티 타입
초기화 과정에서 값을 초기화하지 않아도 되는, 즉 인스턴스가 사용되는 동안에 값을 꼭 갖지 않아도 되는 저장 프로퍼티가 있다면 해당 프로퍼티를 **옵셔널**로 선언할 수 있다. 또는 초기화 과정에서 값을 지정해주기 어려운 경우 저장 프로퍼티를 옵셔널로 선언할 수도 있다.   
옵셔널로 선언한 저장 프로퍼티는 초기화 과정에서 값을 할댕해주지 않는다면 자동으로 `nil`이 할당된다.

~~~swift
// file: "옵셔널 프로퍼티.swift"
class Person {
    var name: String
    var age: Int?   // (옵셔널) 저장 프로퍼티
    
    init(name: String) { self.name = name }
}

let faker = Person(name: "Lee Sang-hyuk")
print(faker.name)   // Lee Sang-hyuk
print(faker.age)    // nil

faker.age = 28
print(faker.age)    // 28

faker.name = "GOAT"
print(faker.name)   // GOAT
~~~

### 상수 프로퍼티
상수로 선언된 프로퍼티는 인스턴스를 초기화하는 과정에서만 값을 할당할 수 있으며, 처음 할당된 이후로는 값을 변경할 수 없다.

> **NOTE_상수 프로퍼티와 상속**
>
> 클래스 인스턴스의 상수 프로퍼티는 프로퍼티가 정의된 클래스에서만 초기화할 수 있다. 해당 클래스를 상속받은 자식클래스의 이니셜라이저에서는 부모클래스의 상수 프로퍼티 값을 초기화할 수 없다.

~~~swift
// file: "상수 프로퍼티의 초기화.swift"
class Person {
    let name: String    // (상수) 저장 프로퍼티
    var age: Int?
    
    init(name: String) { self.name = name }
}

let faker = Person(name: "Lee Sang-hyuk")   // 상수 프로퍼티 초기화
faker.name = "GOAT" // 컴파일 에러!
~~~

### 기본 이니셜라이저와 멤버와이즈 이니셜라이저
기본 이니셜라이저는 저장 프로퍼티의 기본값이 모두 지정되어 있고, 동시에 사용자 정의 이니셜라이저가 정의되어 있지 않은 상태에서 제공된다.

프로퍼티 하나 때문에 매번 이니셜라이저를 추가하거나 변경하는 번거로움을 해결하기 위해 Swift 구조체는 사용자 정의 이니셜라이저를 구현하지 않으면 프로퍼티의 이름으로 매개변수를 갖는 **멤버와이즈 이니셜라이저**를 기본으로 제공한다. 클래스는 멤버와이즈 이니셜라이저를 지원하지 않는다.

~~~swift
// file: "구조체의 멤버와이즈 이니셜라이저 사용.swift"
struct Point {
    // 프로퍼티 기본값 지정
    var x = 0.0
    var y = 0.0
}

struct Size {
    // 프로퍼티 기본값 지정
    var width = 0.0
    var height = 0.0
}

let point = Point(x: 0.0, y: 0.0)
let size = Size(width: 50.0, height: 50.0)

// 구조체의 저장 프로퍼티에 기본값이 있는 경우
// 필요한 매개변수만 사용해서 초기화할 수도 있다.
let somePoint = Point()
let someSize = Size(width: 50.0)
let anotherPoint = Point(y: 100)
~~~

### 초기화 위임
값 타입인 구조체와 열거형은 코드의 중복을 피하기 위해 이니셜라이저가 다른 이니셜라이저에게 일부 **초기화 위임**을 구현할 수 있다. 하지만 클래스는 상속을 지원하기 때문에 간단한 초기화 위임 기능을 지원하지 않는다.

값 타입에서 이니셜라이저가 다른 이니셜라이저를 호출하려면 `self.init`을 사용하며, 이는 이니셜라이저 안에서만 사용할 수 있다. 

사용자 정의 이니셜라이저를 정의하면 기본 이니셜라이저와 멤버와이즈 이니셜라이저를 사용할 수 없으므로 초기화 위임을 하려면 최소 두 개 이상의 사용자 정의 이니셜라이저를 정의해야한다.

> **NOTE_기본 이니셜라이저를 지키고 싶다면**
>
> 사용자 정의 이니셜라이저를 정의할 때도 기본 이니셜라이저나 멤버와이즈 이니셜라이저를 사용하고 싶다면 익스텐션을 사용해 사용자 정의 이니셜라이저를 구현하면 된다.   

~~~swift
// file: "열거형과 초기화 위임.swift"
enum Student {
    case elementary, middle, high
    case none
    
    // 사용자 정의 이니셜라이저가 있는 경우, init() 메서드를 구현해야 기본 이니셜라이저를 사용할 수 있다.
    init() { self = .none }
    
    // 첫 번째 사용자 정의 이니셜라이저
    init(koreanAge: Int) {
        switch koreanAge {
        case 8...13:
            self = .elementary
        case 14...16:
            self = .middle
        case 17...19:
            self = .high
        default:
            self = .none
        }
    }
    
    // 두 번째 사용자 정의 이니셜라이저
    init(bornAt: Int, currentYear: Int) { self.init(koreanAge: currentYear - bornAt + 1) }
}

var younger = Student(koreanAge: 16)
print(younger)  // middle

younger = Student(bornAt: 1998, currentYear: 2016)
print(younger)  // high
~~~

위 코드에서 확인할 수 있듯 초기화 위임 방법을 사용하면 코드를 중복으로 쓰지 않고 효율적으로 여러 `case`의 이니셜라이저를 만들 수 있다.

### 실패 가능한 이니셜라이저
이니셜라이저를 통해 인스턴스를 초기화할 수 없는 여러 가지 예외 상황이 있다. 대표적으로 이니셜라이저의 전달인자로 잘못된 값이나 적절치 못한 값이 전달되었을 때, 이니셜라이저는 인스턴스 초기화에 실패할 수 있다.

이니셜라이저를 정의할 때 이렇게 실패 가능성을 염두에 두기도 하는데, 실패 가능성을 내포한 이니셜라이저를 **실패 가능한 이니셜라이저(Failable initializer)**라고 부른다. 실패 가능한 이니셜라이저는 인스턴스 초기화에 실패했을 때 `nil`을 반환해주므로 반환 타입이 옵셔널로 지정된다. 따라서 실패 가능한 이니셜라이저는 `init?` 키워드를 사용한다.

> **NOTE_이니셜라이저의 매개변수**
>
> 실패하지 않는 이니셜라이저와 실패 가능한 이니셜라이저를 같은 이름과 같은 매개변수 타입을 갖도록 정의할 수 없다.
> 실패 가능한 이니셜라이저는 실제로 특정 값을 반환하지 않는다. 초기화를 실패했을 때는 `return nil`을, 반대로 초기화에 성공했을 때는 `return`만 기입해 초기화의 성공과 실패를 표현할 뿐, 실제 값을 반환하지는 않는다.
>

~~~swift
// file: "실패 가능한 이니셜라이저.swift"
class Person {
    let name: String
    var age: Int?
    
    init?(name: String) {
        if name.isEmpty { return nil }
        
        self.name = name
    }
    
    init?(name: String, age: Int) {
        if name.isEmpty || age < 0 { return nil }
        
        self.name = name
        self.age = age
    }
}

let faker: Person? = Person(name: "Lee Sang-hyuk", age: 28)

if let person = faker {
    print(person.name)  // Lee Sang-hyuk
} else { print("Person wasn't initialized.") }

let keria: Person? = Person(name: "Ryu Min-seok", age: -22)

if let person = keria {
    print(person.name)
} else { print("Person wasn't initialized.") }  // Person wasn't initialized.

let owner: Person? = Person(name: "", age: 21)

if let person = owner {
    print(person.name)
} else { print("Person wasn't initialized.") }  // Person wasn't initialized.
~~~

실패 가능한 이니셜라이저는 열거형에서 유용하게 사용된다. 특정 `case`에 맞지 않는 값이 들어오거나, `rawValue`로 초기화할 때 잘못된 `rawValue`가 전달되면 열거형 인스턴스를 생성하지 못할 수 있다. 그러므로 `rawValue`를 통한 이니셜라이저는 기본적으로 실패 가능한 이니셜라이저로 제공된다.

~~~swift
// file: "열거형의 실패 가능한 이니셜라이저.swift"
enum Student: String {
    case elementary = "초등학생"
    case middle = "중학생"
    case high = "고등학생"
    
    init?(koreanAge: Int) {
        switch koreanAge {
        case 8...13:
            self = .elementary
        case 14...16:
            self = .middle
        case 17...19:
            self = .high
        default:
            return nil
        }
    }
    
    init?(bornAt: Int, currentYear: Int) { self.init(koreanAge: currentYear - bornAt + 1) }
}

var younger = Student(koreanAge: 20)
print(younger)  // nil

younger = Student(bornAt: 2020, currentYear: 2016)
print(younger)  // nil

younger = Student(rawValue: "대학생")
print(younger)  // nil

younger = Student(rawValue: "고등학생")
print(younger)  // Optional(Student.high)
~~~

### 함수를 사용한 프로퍼티 기본값 설정
사용자 정의 연산을 통해 프로퍼티 기본 값을 설정하고자 한다면 클로저나 함수를 사용해 프로퍼티 기본값을 제공할 수 있다. 인스턴스를 초기화할 때 함수나 클로저가 호출되면서 연산 결과값을 프로퍼티 기본값으로 제공하는 매커니즘이다. 따라서 ***클로저나 함수의 반환 타입은 프로퍼티의 타입과 일치해야 한다.***   
또한 클로저를 사용해 프로퍼티 기본 값을 설정한다면, ***클로저가 실행되는 시점은 초기화할 때 인스턴스의 다른 프로퍼티 값이 설정되기 전이다!*** 즉, 클로저 내부에서는 인스턴스의 다른 프로퍼티를 사용해서 연산할 수 없으며, 다른 프로퍼티의 기본값이 존재해서도 안된다. 또한, 클로저 내부에서 `self` 프로퍼티도 사용할 수 없으며, 인스턴스 메서드 호출도 불가능하다.

~~~swift
// file: "클로저를 통한 프로퍼티 기본값 설정.swift"
class SomeClass {
    let somProperty: SomeType = {
        // 새로운 인스턴스를 생성하고 사용자 정의 연산을 통한 후 반환해준다.
        // 반환되는 값의 타입은 SomeType과 같은 타입이어야 한다.
        return someValue
    }()
}
~~~

위 코드에서 크로저 뒤에 소괄호가 붙으며, 클로저를 실행한 결과값은 프로퍼티의 기본값이 된다. 만약 소괄호가 없다면 프로퍼티의 기본값은 클로저 그 자체가 된다. 

~~~swift
// file: "클로저를 통한 프로퍼티 기본값 설정.swift"
struct Student {
    var name: String?
    var number: Int?
}

class SchoolClass {
    var students: [Student] = {
        // 새로운 인스턴스를 생성하고 사용자 정의 연산을 통한 후 반환해준다.
        // 반환되는 값의 타입은 [Student] 타입이어야 한다.
        
        var arr = [Student()]
        
        for num in 1..<15 {
            var student = Student(name: nil, number: num)
            arr.append(student)
        }
        
        return arr
    }()
}

let myClass = SchoolClass()
print(myClass.students.count)   // 15
~~~

> **Tip. iOS에서 활용**
>
> iOS의 UI등을 구성할 때, UI 컴포넌트를 클래스의 프로퍼티로 구현하고, UI 컴포넌트의 생성과 동시에 컴포넌트의 프로퍼티를 기본적으로 설정할 때 유용하게 사용
>

## 인스턴스 소멸
클래스의 인스턴스는 **디이니셜라이저(Deinitializer)**를 구현할 수 있다. 디이니셜라이저는 이니셜라이저와 반대 역할을 수행하며, 메모리에서 해제되기 직전 클래스 인스턴스와 관련하여 원하는 정리 작업을 구현할 수 있다.   
디이니셜라이저는 클래스의 인스턴스가 메모리에서 소멸되기 직전에 호출되며 `deinit` 키워드를 사용해 구현해두면 자동으로 호출된다.

Swift는 인스턴스가 더 이상 필요하지 않으면 ARC를 통해 메로리에서 소멸시키므로 인스턴스 대부분은 소멸시킬 때 디이니셜라이저를 사용해 별도의 메모리 관리 작업을 할 필요 없다.

클래스에는 디 이니셜라이저를 하나만 구현할 수 있다. 디이니셜라이저는 매개변수와 소괄호를 갖지 않으며, 자동으로 호출되기 때문에 별도의 코드로 호출할 수도 없다.

~~~swift
// file: "디이니셜라이저 구현.swift"
class SomeClass {
    deinit { print("Instance will be deallocated immediately") }
}

var instance: SomeClass? = SomeClass()
instance = nil  // Instance will be deallocated immediately
~~~
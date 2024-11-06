---
layout: post
title: "Swift #Section05 - ARC"
sitemap: false
image: /assets/img/blog/thumbnail/swift/swift_section04_closure.png
categories: [study, swift]
tags: [swift]
description: 본 글에서는 클로저의 다양한 형태 및 표현법과 탈출, 자동 클로저에 대해 다룬다.
---

* toc
{:toc}

## 🤚 Introduce
Reference Type은 하나의 인스턴스가 참조(메모리 주소)를 통해 여러 곳에 엑세스하기 때문에 언제 메모리에서 해제되는지가 중요하다.
인스턴스가 적절한 시기에 메모리에서 해제되지 않으면 한정적인 메모리와 자원을 낭비하고 이는 곧 메모리 누수로 이어진다.
Swift는 프로그램의 메모리 사용을 관리하기 위해 **ARC(Automatic Reference Counting)**을 지원한다.

## ARC란?
ARC는 더이상 필요하지 않은 클래스의 인스턴스를 메모리에서 해제하는 방식으로 동작한다.
Java의 가비지 컬렉션과 달리 ARC는 컴파일과 동시에 인스턴스를 메모리에서 해제하는 시점이 결정된다.
따라서 원하는 방향으로 메모리 관리가 이루어지려면 ARC에 명확한 힌트를 주어야 한다.

Reference Tpye의 인스턴스를 생성할 때마다 ARC는 해당 인스턴스에 대한 정보를 처리하기 위해 별도의 메모리 공간을 별로도 할당한다. 
해당 메모리 공간에는 인스턴스의 타입 정보와 함께 인스턴스와 관련된 저장 프로퍼티의 값 등을 저장한다.

## 강한참조
인스턴스를 다른 인스턴스의 프로퍼티나 변수, 상수 등에 할당할 때 강한 참조를 사용하면 RC가 1 증가한다.
또한 강한 참조를 사용하면 프로퍼티 변수, 상수 등에 nil을 할당할 해주면 인스턴스에 할당된 RC가 1 감소한다.

참조의 기본은 강한참조이므로 클래스 타입의 프로퍼티, 변수, 상수 등을 선언할 때 별도의 식별자를 명시하지 않으면 강한참조를 한다.
~~~swift
// file: "Code01.swift"
class Person {
    let name: String
    
    init(name: String) {
        self.name = name
        print("\(name) is being initialized")
    }
    
    deinit {
        print("\("name") is being deinitialized")
    }
}

var reference01: Person?
var reference02: Person?
var reference03: Person?

reference01 = Person(name: "Faker")
// Faker is being initialized
// Person instance RC: 1

reference02 = reference01   // Person instance RC: 2
reference03 = reference01   // Person instance RC: 3

reference03 = nil           // Person instance RC: 2
reference02 = nil           // Person instance RC: 1
reference01 = nil           // Person instance RC: 0
// Faker is being deinitialized
~~~

Code01에서 `Person` 인스턴스의 데이터가 Heap 영역에 할당된 후 `reference01`변수는 Call by reference를 통해 `Person`인스턴스의 참조 주소를 획득한다. 이때 강한 참조가 발생하여 `Person` 인스턴스의 RC가 1증가한다.
이후 `reference02`, `reference03`는 `reference01`이 갖고있는 `Person` 인스턴스의 참조를 획득하면서 `Person` 인스턴스의 RC이 증가하여 최종적으로 `Person` 인스턴스의 RC는 3이 된다.

또한 `Person`인스턴스를 참조하는 각 변수에 `nil`을 할당하면서 `Person` 인스턴스의 RC가 1씩 감소하고 되고 최종적으로 RC가 0이 됐을 때 `Person`의 인스턴스가 Heap 영역에서 해제되면서 디이니셜라이저가 호출된다.

~~~swift
// file: "Code02.swift"
var globalReference: Person?

func foo() {
    let faker = Person(name: "Faker")   // Person instance RC: 1
    // Faker is being initialized

    globalReference = faker // Person instance RC: 2
    
    // 함수 종료 시점
    // Person instance RC: 1
}

foo()
~~~

Code02에서 함수가 실행되면 `Person` 인스턴스의 참조가 할당된 지역상수가 존재하며 이때 `Person` 인스턴스는 RC는 1이다.
또한 함수 내부에서 전역변수에 Call by reference를 통해 `Person` 인스턴스의 참조를 할당해주면서 `Person` 인스턴스 RC는 2가 된다.
함수가 종료되면 해당 지역상수가 참조하던 `Person` 인스턴스의 RC가 1 감소하지만 전역변수가 갖은 `Person` 인스턴스의 참조는 해제되지 않았으므로 최종적으로 `Person` 인스턴스 RC는 1이되어 Heap 영역에서 해제되지 않는다.

### 강한 참조 순환 문제
인스턴스끼리 서로가 서로를 강한 참조할 때 `강한참조 순환`이라 한다.
~~~swift
// file: "Code03.swift"
class Person {
    let name: String
    var room: Room?
    
    init(name: String) {
        self.name = name
    }
    
    deinit {
        print("\(name) is being deinitialized")
    }
}

class Room {
    let number: Int
    var host: Person?
    
    init(number: Int) {
        self.number = number
    }
    
    deinit {
        print("Room \(number) is being deinitialized")
    }
}

var somePerson: Person? = Person(name: "person") // Person instance RC: 1
var someRoom: Room? = Room(number: 100)       // Room instance RC: 1


someRoom?.host = somePerson  // Person instance RC: 2
somePerson?.room = someRoom  // Room instance RC: 2

somePerson = nil    // Person instance RC: 1
someRoom = nil      // Room instanceh RC: 1
~~~

## 아래 내용부터는 정리해서 다시 업로드
/*
 - Value Type과 달리 Reference Type의 인스턴스는 참조(주소)를 통해 여러곳에서 Heap 영역에 위치한 인스턴스에 엑세스할 수 있다. - Call by Reference
 - 참조를 통해 Heap영영게 위치한 인스턴스에 접근할 수 있으므로 언제 참조하고 있는 인스턴스가 Heap 영역에서 에서 해제되는지가 관건이다.
    - 더이상 필요하지 않은 인스턴스가 Heap 영역에서 해제되지 않으면 메모리 누수 발생!!
 - Swift는 한정적인 메모리 자원을 관리하기 위해 메모리 관리 기법인 ARC(Automatic Reference Counting)을 지원
*/


/*
 <강한 참조>
 - 인스턴스 참조를 다른 인스턴스의 프로퍼티나 변수등에 할당할 때 강한 참조가 발생하며, 해당 인스턴스의 RC가 1 증가한다.
 - 강한 참조를 사용하는 프로퍼티나 변수 등에 nil을 할당하면 RC가 1 감소한다.
 - 인스턴스의 RC가 0이 되는 순간 해당 인스턴스는 Heap 영역에서 해제된다.
 
 - 참조의 기본은 강한 참조!!
    - 프로퍼티나, 변수 등에 참조를 할당할 때 별도의 식별자를 명시하지 않으면 강한 참조가 발생
 */


class Person {
    let name: String
    
    init(name: String) {
        self.name = name
        print("\(name) is being initialized.")
    }
    
    deinit { print("\(name) is being deinitialized.") }
}

var reference01: Person?
var reference02: Person?
var reference03: Person?

reference01 = Person(name: "Faker")     // Person instance RC: 1
reference02 = reference01               // Person instance RC: 2
reference03 = reference01               // Person instance RC: 3

reference03 = nil               // Person instance RC: 2
reference02 = nil               // Person instance RC: 1
reference01 = nil               // Person instance RC: 0

// deinit { print("\(name) is being deinitialized.") }







/*
 <강한 참조 순환 문제>
 - 인스턴스끼리 서로 강한 참조할 때 발생!!
 */

class Person01 {
    let name: String
    var room: Room01?
    
    init(name: String) {
        self.name = name
    }
    
    deinit { print("\(name) is being deinitialized.") }
}

class Room01 {
    let number: Int
    var host: Person01?
    
    init(number: Int) {
        self.number = number
    }
    
    deinit { print("Room \(number) is being deinitialized.") }
}

var faker: Person01? = Person01(name: "Lee Sang-hyuk")  // Person01 instance RC: 1
var room: Room01? = Room01(number: 100)                // Room01 instance RC: 1

faker?.room = room  // Room01 instance RC: 2
room?.host = faker  // Person01 instance RC: 2

faker = nil // Person01 instance RC: 1
room = nil  // Room01 instance RC: 1











/*
 강한 순환 참조 수동 해결
 */

faker?.room = nil   // Room01 instance RC: 1
faker = nil         // Person01 instance RC: 1

room?.host = nil    // Person01 instance RC: 0
// Lee Sang-hyuk is being deinitialized.

room = nil          // Room01 instance RC: 0
// Room 100 is being deinitialized.






/*
 <약한 참조>
 - 프로퍼티나 변수에 인스턴스의 참조를 할당할 때 참조하는 인스턴스의 RC를 증가시키지 않는다.
 - weak 키워드를 프로퍼티나 변수 앞에 명시하면 해당 프로퍼티는 자신이 참조하는 인스턴스를 약한 참조하게 설정된다.
 
 - 약한 참조를 한다는 것은 해당 인스턴스가 Heap에서 해제될 수 있다는 것을 예상해야 한다.
    - 자신이 RC를 증가시키지 않았으므로, 해당 인스턴스를 강한 참조하는 프로퍼티가 RC를 감소시켜 0으로 만들면
      자신이 참조하던 인스턴스는 해제됨
 
 - 약한 참조 -> 상수? 옵셔널?
    - 참조하던 인스턴스가 Heap 에서 해제된다면 nil이 할당 될 수 있어야 하므로 변수 이면서 옵셔널로 선언해야 한다.
    - weak var
 */
class Person02 {
    let name: String
    var room: Room02?
    
    init(name: String) {
        self.name = name
    }
    
    deinit { print("\(name) is being deinitialized.") }
}

class Room02 {
    let number: Int
    weak var host: Person02?
    
    init(number: Int) {
        self.number = number
    }
    
    deinit { print("Room \(number) is being deinitialized.") }
}

var keria: Person02? = Person02(name: "Keria")   // Person02 instance RC: 1
var keriaRoom: Room02? = Room02(number: 100)     // Room02 instance RC: 1


keria?.room = keriaRoom // Room02 instance RC: 2
keriaRoom?.host = keria // Person02 instance RC: 1

keria = nil
// Person02 instance RC: 0
// Keria is being deinitialized
// Room02 instance RC: 1


keriaRoom = nil
// Room02 instance RC: 0
// Room 100 is being deinitialized.







/*
 <미소유 참조>
 - 약한 참조와 마찬가지로 미소유 참조는 인스턴스의 RC를 증가시키지 않는다.
 - 약한 참조와는 다르게 참조하는 인스턴스가 항상 Heap에 존재할 것이라는 전제를 기반으로 동작한다.
    - 인스턴스가 Heap에서 해제돼도 스스로 nil을 할당해주지 않는다.
    - 미소유 참조를 할당받는 변수나 프로퍼티는 옵셔널이나 변수가 아니어도 된다.
 
 - But 미소유 참조를 하면서 Heap에서 해제된 인스턴스에 접근하려 하면 잘못된 접근으로 런타임 에러 발생.
    - 자신이 참조하는 인스턴스가 메모리에서 해제되지 않으리라는 확신이 있을 때만 사용해야한다.
 
 - unowned 키워드 명시를 통해 미소유 참조 설정 가능
 */

class Person03 {
    let name: String
    var card: CreditCard?
    
    init(name: String) {
        self.name = name
    }
    
    deinit { print("\(name) is being deinitialized")}
}

class CreditCard {
    let number: UInt
    unowned let owner: Person03
    
    init(number: UInt, owner: Person03) {
        self.number = number
        self.owner = owner
    }
    
    deinit { print("Card #\(number) is being deinitialized.") }
}

var zeus: Person03? = Person03(name: "Zeus")    // Person03 instance RC: 1

if let person = zeus {
    person.card = CreditCard(number: 10000, owner: person)
    // CreditCard instance RC: 1
    // Person03 instacnce RC: 1
}

zeus = nil
// Person03 instance RC: 0
// CreditCard instance RC: 0 <- Person03의 card 프로퍼티가 강한 참조하던 CreditCard RC도 같이 감소하는 것을 확인할 수있음







/*
 <미소유 옵셔널 참조>
 Refercnce Type을 참조하는 옵셔널을 미소유로 표시할 수 있다.
 ARC 소유 모델에 따른 미소유 옵셔널 참조와 약한 참조를 같은 상황에서 사용할 수 있다.
    - 미소유 옵셔널 참조는 항상 유요한 객체를 가리키거나 그렇지 않으면 nil을 할당해 주도록 직접 신경을 써야 한다.
 */

class Department {
    var name: String
    var subjects: [Subject] = []
    
    init(name: String) {
        self.name = name
    }
}

class Subject {
    var name: String
    unowned var department: Department
    unowned var nextSubject: Subject?
    
    init(name: String, in department: Department) {
        self.name = name
        self.department = department
        self.nextSubject = nil
    }
}

let department = Department(name: "CS")

let intro = Subject(name: "Architecture", in: department)
let intermediate = Subject(name: "Swift Language", in: department)
let advanced = Subject(name: "iOS", in: department)

intro.nextSubject = intermediate
intermediate.nextSubject = advanced
department.subjects = [intro, intermediate, advanced]










/*
 <클로저의 강한 순환 참조>
 - 클로저가 인스턴스의 프로퍼티거나 클로저의 값 획득 특성 때문에도 강한 순환 참조가 발생할 수 있다.
    - 클로저 내부에서 self.someProperty 처럼 인스턴스의 프로퍼티에 접근할 때
    - 클로저 내무에서 self.someMethod() 처럼 인스턴스의 메서드를 호출할 때. 값 획득이 발생한다.
    - 두 사례 모두 self 프로퍼티 참조를 획득하므로 강한 순환 참조가 발생한다.
 
 - 강한 순환 참조가 발생하는 이유는 클로저가 Reference Type이기 때문이다.
    - 클로저를 클래스 인스턴스의 프로퍼티로 할당하면 클로저의 참조가 할당된다. 이때 Class의 참조와 클로저의 참조가 서로 강한 참조를 하면서 강한 참조가 발생한다.
*/

class Person04 {
    let name: String
    let hobby: String?
    
    // 클로저는 호출되면 자신 내부에 있는 참조들을 사용할 수 있도록 해당 참조 인스턴스 RC를 증가시켜 메모리에서 해제되는 것을 방지한다.
    lazy var introduce: () -> String = {
        var introduction = "My name is \(self.name)"
        
        guard let hobby = self.hobby else { return introduction }
        
        introduction += " "
        introduction += "My hobby is \(hobby)"
        
        return introduction
    }
    
    init(name: String, hobby: String? = nil) {
        self.name = name
        self.hobby = hobby
    }
    
    deinit { print("\(name) is being deinitialized.") }
}

var seongGyu: Person04? = Person04(name: "seongGyu", hobby: "game") // Person04 instance RC: 1
print(seongGyu?.introduce() ?? "")    // Person04 instance RC: 2
seongGyu = nil // deinit 호출되지 않음!!!! - 메모리 누수






/*
 <획득 목록>
 - 캡처 리스트는 클로저 내부에서 참조 타입을 획득하는 규칙을 제시해주는 기능이다.
 - 획특목록에 명시한 요소가 Reference Type이 아니라면 해당 요소들은 클로저가 생성될 때 초기화 된다.
 */

var a = 0
var b = 0

let clousre = { [a] in
    print(a, b)
    b = 20
}

a = 10
b = 10

clousre()   // 0, 10
// 획득 목록에 명시한 요소가 Referecnce Type이 아니라면 해당 요소들은 클로저가 생성될 때 상수로 초기화된다.
// 외부에서 a 값을 변경해도 클로저의 획득 목록을 통한 a와는 별개가 된다.
print(b)    // 20



class SimpleClass {
    var value = 0
}

var x = SimpleClass()
var y = SimpleClass()

x.value = 10
y.value = 10

let clousre01 = { [x] in
    print(x.value, y.value)
}

clousre01() // 10, 10

// 캠처 목록의 요소가 참조타입이므로 획득목록에서 제외된 다른 참조 타입과 같은 결과 확인
// 참조 타입 획득시 어떤 방식으로 참조할 것인지, 획득 종류에 따라 RC를 증가시킬지 결정할 수 있다.
//  - 약한 획득을 하게되면 획득 목록에서 획득하는 상수가 옵셔널 지정
//      -> 클로저 내부에서 약한 획득한 상수를 사용하려고 할 때 이미 메모리에서 해제된 상태일 수 있기 때문
//      -> 해제된 후 접근하려 하면 잘못된 접근으로 오류가 발생하므로 안전을 위해 약한 획득은 기본적으로 타입을 옵셔널로 사용

class SimpleClass01 {
    var value: Int = 0
}

var x01: SimpleClass01? = SimpleClass01()
var y01 = SimpleClass01()

let closure01 = { [weak x01, unowned y01] in
    print(x01?.value, y01.value)
}

x01 = nil   // x01가 참조하는 인스턴스가 메모리에서 해제되면 클로저 내부에서도 더이상 참조 불가능
y01.value = 10

closure01() // nil 10


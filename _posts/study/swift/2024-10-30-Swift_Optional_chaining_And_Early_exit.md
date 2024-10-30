---
layout: post
title: "Swift Optional chaining And Early Exit"
sitemap: false
image: /assets/img/blog/thumbnail/swift/swift_structureAndClasses.png
categories: [study, swift]
tags: [swift]
description: 본 글에서는 Swift의 대표적인 사용자 정의 데이터 구조체와 클래스에 대해 다룬다.
---

* toc
{:toc}

## Intro
**옵셔널 체이닝(Optional Chainig)**은 여러 값이 중첩된 형태에서 제 몫을 발휘한다. 해당 글에서 옵셔널을 좀 더 편리하게 사용할 수 있는 옵셔널 체이닝과 **빠른 종료(Early Exit)**에 대해 알아본다.

## 옵셔널 체이닝
옵셔널 체이닝은 옵셔널에 속해 있는 `nil`일지 모르는 프로퍼티, 메서드, 서브스크립트 등을 가져오거나 호출할 때 사용할 수 있는 일련의 과정이다. 옵셔널 체이닝은 데이터를 호출하고 싶은 옵셔널 변수나 상수 뒤에 `?`를 붙여 표현하며, 옵셔널 값이 존재한다면 해당 데이터를 호출할 수 있고, 옵셔널이 `nil`이라면 해당 데이터는 `nil`을 반환한다.

중첩된 옵셔널 중 하나라도 값이 존재하지 않는다면 `nil`을 반환하며, 결과적으로 `nil`이 반환될 가능성이 있으므로 ***옵셔널 체이닝의 반환된 값은 항상 옵셔널이다.***

<br>

~~~swift
// file: "Code01.swift"
class Room {
    var number: Int
    
    init(number: Int) {
        self.number = number
    }
}

class Building {
    var name: String
    var room: Room?
    
    init(name: String) {
        self.name = name
    }
}

struct Address {
    var province: String
    var city: String
    var street: String
    var builing: Building?
    var detailAddress: String?
}

class Person {
    var name: String
    var address: Address?
    
    init(name: String) {
        self.name = name
    }
}

let faker = Person(name: "Lee Sang-hyuk")

let fakerRoomViaOptionalChaining = faker.address?.builing?.room?.number
// nil

let fakerRoomViaOptionalUnwraping = faker.address!.builing!.room!.number
// 런타입 에러!
~~~

Code01에서 확인해 볼 수 있듯 `faker` 인스턴스에는 아직 주소, 건물, 호실 정보가 없으므로 `fakerRoomViaOptionalChaing` 상수에 호실 번호를 할당하려고 옵셔널 체이닝을 사용하면 `nil`을 반환받게 된다. 반면 `fakerRoomViaUmwrapping` 상수에 데이터를 할당하려 할때 강제 추출을 시도했으며, `nil`인 프로퍼티에 접근했으므로 런타임 에러가 발생한 것을 확인할 수 있다.

<br>

~~~swift
// file: "Code02.swift"
let faker = Person(name: "Lee Sang-hyuk")

var roomNumber: Int? = nil

if let fakerAddress = faker.address {
    if let fakerBuilding = fakerAddress.builing {
        if let fakerRoom = fakerBuilding.room {
            roomNumber = fakerRoom.number
        }
    }
}

if let number = roomNumber {
    print(number)
} else {
    print("Can not find room number")
}
~~~
~~~swift
// file: "Code03.swift"
let faker = Person(name: "Lee Sang-hyuk")

if let roomNumber = faker.address?.builing?.room?.number {
    print("roomNumber")
} else {
    print("Can not find room number")
}
~~~
Code02는 옵셔널 바인딩을 사용해 옵셔널 데이터를 가져오는 코드이며, Code03은 옵셔널 체이닝을 사용해서 훨씬 간단한 로직으로 구성된 것을 확인할 수 있다. 두 코드의 결과는 같지만 간결성과 분량의 차이가 존재한다. 

또한 Code03에서 옵셔널 바인딩과 옵셔널 체이닝이 결합한 것을 확인할 수 있는데 ***옵셔널 체이닝의 결과값은 옵셔널 값이기 떄문에 옵셔널 바인딩과 결합할 수 있는 것이다.***

<br>

옵셔널 체이닝을 통해 값을 받아오기만 하는 것이 아니라 반대로 값을 할당해줄 수도 있다.
~~~swift
// file: "Code04.swift"
faker.address = Address(province: "Seoul", city: "Gang-Nam", street: "Dosan street")
faker.address?.builing = Building(name: "Faker tower")
faker.address?.builing?.room = Room(number: 999)
faker.address?.builing?.room?.number = 505

print(faker.address?.builing?.room?.number) // Optional(505)
~~~

---

## 빠른 종료
**빠른 종료(Early Exit)**에 사용되는 `gurad` 구문은 `if` 구문과 유사하게 `Bool` 타입의 값으로 동작하며, `guard` 뒤 코드 결과에 따라 `true`일 때 코드가 계속 실행된다.

`guard` 구문은 항상 `else` 구문이 뒤에 붙으며, `guard` 뒤 코드의 결과가 `false`일때 `else` 블록 내부의 코드가 실행된다. 이때 ***`else` 블록 내부에는 `guard` 구문을 실행하는 코드보다 상위의 코드 블록을 종료하는 코드가 들어가야 한다.*** 따라서 `guard` 구문의 조건에 부합하지 않다는 판단이 되면 빠르게 코드 블록의 실행을 종료할 수 있다. 

~~~swift
// file: "guard 구문 정의.swift"
guard ValueOfBoolType else {
    // Exception Code ...
    // return, break, continue, throw, fatalError ...
}
~~~

### guard vs if
`guard` 구문을 사용하면 `if` 구문을 간결하고 가독성있게 구성할 수 있다. `if` 구문을 사용하면 예외사항을 `else` 블록으로 처리하지만 예외사항만을 처리하고 싶다면 `guard` 구문을 사용하는 것이 더 간편하다.

~~~swift
// file: "Code05.swift"
// if 구문을 사용한 코드
for i in 0...3 {
    if i == 2 {
        print(i)
    } else {
        continue
    }
}

// guard 구문을 사용한 코드
for i in 0...3 {
    guard i == 2 else { continue }
    
    print(i)
}
~~~

### guard 구문의 옵셔널 바인딩 활용
`guard` 구문을 통해 옵셔널 바인딩의 역할을 수행할 수 있다. `guard` 뒤에 위치하는 옵셔널 바인딩 표현에서 옵셔널의 값이 `nil`이 아니라면 `guard` 구문에서 옵셔널 바인딩된 상수 및 변수를 `guard` 구문이 실행된 아래 코드부터 지역상수처럼 사용할 수 있다.

~~~swift
// file: "Code06.swift"
func greet(_ person: [String: String]) {
    guard let name = person["name"] else { return }
    
    print("Hello \(name)!")
    
    guard let location = person["location"] else {
        print("I hope the weather is nice near you")
        
        return
    }
    
    print("I hope the weahter is nice in \(location)")
}

var personInfo = [String: String]()
personInfo["name"] = "Faker"
greet(personInfo)
// Hello Faker!
// I hope the weather is nice near you

personInfo["location"] = "Seoul"
greet(personInfo)
// Hello Faker!
// I hope the weahter is nice in Seoul
~~~

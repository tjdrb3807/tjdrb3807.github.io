---
layout: post
title: "Swift Optionals"
sitemap: false
image: /assets/img/blog/thumbnail/swift/swift_instance.png
categories: [study, swift]
tags: [swift]
description: 본 글에서는 Swift 사용자 지정 데이터의 인스턴스 생성 및 소멸에 대해 다룬다.
---

* toc
{:toc}

## 🤚 Introduce
**옵셔널(Optionals)**은 안전성을 문법으로 담보하는 기능이며, 변수나 상수 등에 값이 있다는 것을 보장할 수 없음을 의미한다. 즉, 변수 또는 상수의 값이 `nil`일 수도 있다는 것을 뜻한다.


## 옵셔널 사용
옵셔널 사용은 변수 또는 상수가 `nil`일 수도 있으므로 사용에 주의하라는 뜻으로 직관적으로 받아들일 수 있다. 값이 없는 옵셔널 변수 또는 상수에 (강제로) 접근하게되면 런타임 에러가 발생한다. 

옵셔널 변수 또는 상수 등은 데이터 타입 뒤에 `?`를 붙여 표현한다.

~~~swift
// file: "Code01.swift"
var name: String? = "Faker"
print(name) // Optional("Faker")

name = nil
print(name) // nil
~~~

<br>

옵셔널은 제네릭이 적용된 열거형이다. 

~~~swift
// file: "Optional.swift"
@frozen public enum Optional<Wrapped> : ~Copyable where Wrapped : ~Copyable {
    // ...

    case none
    case some(Wrapped)

    public init(_ some: consuming Wrapped)

    //...
}
~~~

위 코드는 Swift의 Optional 코드의 일부이다. 코드에서 확인할 수 있듯 옵셔널은 값을 갖는 케이스와 그렇지 못한 케이스 두 가지로 정의되어 있다. 즉, `nil` 일 때는 `none` 케이스가 될 것이고, 값이 있는 경우 `some`케이스가 되며 `Wrapped`에 연관 값이 할당돼 옵셔널이라는 열거형에 값이 래핑되어 있는 모습이다.

~~~swift
// file: "Code01.swift"
let numbers: [Int?] = [2, nil, -4, nil, 100]

for number in numbers {
    switch number {
    case .some(let value) where value < 0:
        print("Negative value!! \(value)")
    case .some(let value) where value > 10:
        print("Large value!! \(value)")
    case .some(let value):
        print("Value \(value)")
    case .none:
        print("nil")
    }
}

//Value 2
//nil
//Negative value!! -4
//nil
//Large value!! 100
~~~

## 옵셔널 추출
하나의 옵셔널을 `switch` 구문으로 매번 값의 유무를 확인하는 것은 번거롭다. Swift는 옵셔널 타입에서 값을 안전하고 편리하게 추출하는 방법을 제공한다.

### 강제 추출
**옵셔널 강제 추출(Forced Unwrapping)** 방식은 옵셔널의 값을 추출하는 가장 방법이다. 하지만 간단하면서도 가장 위헙한 방법이다. 

옵셔널 값을 강제 추출하기 위해 옵셔널 값 뒤에 `!`를 붙여주면 값을 강제로 추출하여 반환해준다. 하지만 강제 추출 시 옵셔널에 값이 없다면, 즉 `nil` 이라면 런타임 에러가 발생한다.

~~~swift
// file: "Code02.swift"
var name: String? = "Faker"

var faker: String = name!

name = nil
faker = name!   // 런타임 에러! - 옵셔널 값이 nil

// if 구문 등 조건문을 이용해 조금 더 안전하게 처리해볼 수 있다.
if name != nil {
    print("My name is \(name!)")
} else {
    print("name == nil" )
}
// name ==  nil
~~~

### 옵셔널 바인딩
`Code02`에서 if 문을 통해 옵셔널 값의 nil 유무를 판단하고 강제 추출 하는 방법은 옵셔널 사용 의미에 맞지 않다. Swift는 더 안전하고 간결한 방법으로 **옵셔널 바인딩(Optional Binding)**을 제공한다.

옵셔널 바인딩은 옵셔널에 값이 있는지 확인할 때 사용하며, 옵셔널에 값이 있다면 옵셔널에서 추출한 값을 일정 블록 안에서 사용할 수 있는 상수나 변수로 할당해 옵셔널이 아닌 형태로 사용할 수 있도록 지원한다.

옵셔널 바인딩은 `if` 또는 `while` 구문 등과 결합하여 사용할 수 있다.

~~~swift
// file: "Code03.swift"
var name: String? = "Faker"

if let name = name {
    print("My name is \(name)")
} else {
    print("name == nil")
}

// My name is Faker

if var name = name {
    name = "Keria"
    print("My name is \(name)")
} else {
    print("name == nil")
}

// My name is Keria
~~~

`,`를 사용해 바인딩 할 옵셔널을 나열하면, 한 번에 여러 옵셔널 값을 추출할 수도 있다. 이떄, 바인딩하려는 옵셔널 중 하나다로 값이 없다면 해당 블록 내부의 명령문은 실행되지 않는다.

~~~swift
// file: "Code.04.swift"
var myName: String? = "Faker"
var yourName: String? = nil

// colleague == nil 이므로 실행되지 않는다.
if let name = myName, let colleague = yourName {
    print("We are T1! \(name) & \(colleague)")
}

yourName = "Keria"

if let name = myName, let colleague = yourName {
    print("We are T1! \(name) & \(colleague)")
}

// We are T1! Faker & Keria
~~~

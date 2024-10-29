---
layout: post
title: "Swift Structures and Classes"
sitemap: false
image: /assets/img/blog/thumbnail/swift/swift_structureAndClasses.png
categories: [study, swift]
tags: [swift]
description: 본 글에서는 Swift의 대표적인 사용자 정의 데이터 구조체와 클래스에 대해 다룬다.
---

* toc
{:toc}

## Intro
구조체와 클래스는 사용자 정의 데이터 타입을 생성한다.  
구조체의 인스턴스는 값 타입(Value Type)이고 클래스의 인스턴스는 참조 타입(Reference Type)이다.   
Swift의 데이터 타입과 열거형은 모두 값 타입이다.    

## Struct
### Definition
구조체는 `struct` 키워드로 정의하며, 괄호안에 구조체의 정의가 위치한다.

~~~swift
struct SomeStructure {
    // structure definition goes here
}
~~~

구조체를 정의한다는 것은 새로운 타입을 생성해주는 것과 마찬가지이므로 기본 타입 이름처럼 대문자 카멜케이스를 사용하여 이름을 지어준다.
{:.figcaption}

### Instances
구조체를 정의한 후 인스턴스를 생성하고 초기화하고자 할 때는 기본적을 생성되는 멤버와이즈 이니셜라이저를 제공하며, 기본 생성된 이니셜라이저의 매개변수는 구조체의 프로퍼티 이름으로 자동 지정된다.   

인스턴스가 생성되고 초기화된 후 프로퍼티 값에 접근하고 싶으면 마침표(`.`)를 사용한다.   

또한 구조체를 상수 `let`으로 선언하면 인스턴스 내부의 프로퍼티 값을 변경할 수 없고, 변수 `var`로 선언하면 내부의 프로퍼티가 `var`로 선언된 경우에 값을 변경할 수 있다.

~~~swift
var fakerInfo = BasicInformation(name: "Lee Sang hyuk", age: 28)
fakerInfo.age = 100
fakerInfo.name = "GOAT"
~~~
인스턴스를 변수로 선언하여 내부 저장 프로퍼티 변경 가능
{:.figcaption}

~~~swift
let ownerInfo = BasicInformation(name: "Moon Hyun joon", age: 21)
ownerInfo.age = 200 // Comilation Error!
~~~
인스턴스를 상수로 선언하여 내부 저장 프로퍼티 변경시 컴파일 에러 발생
{:.figcaption}

## Class 
### Definition
클래스는 `class` 키워드로 정의한다. 

~~~swift
class SomeClass {
    // class definition goes here
}
~~~

클래스를 정의하는 방법은 구조체와 비슷하지만, 클래스는 상속받을 수 있기 때문에 상속 받을 때는 클래스 이름 뒤에 콜론(`:`)을 써주고 부모클래스 이름을 명시한다.

~~~swift
class SomeClasee: SuperClass {
    // class definition goes here
}
~~~

### Instances
클래스도 구조체와 마찬가지로 인스턴스를 생성하고 초기화하고자 할 때 기본적인 이니셜라이저를 제공한다. 

클래스의 인스턴스는 __참조 타입__ 이므로 인스턴스를 상수 `let`으로 선언해도 내부 프로퍼티 값을 변경할 수 있다.

~~~swift
class Person {
    var heigth: Float = 0.0
    var weight: Float = 0.0
}

let faker = Person()
faker.heigth = 176.0
~~~

### Destruction of instances
클래스의 인스턴스는 __참조 타입__ 이므로 해당 인스턴스가 더이상 참조할 필요가 없을 때 메모리에서 해제된다. 위 과정을 소멸이라 하며, 인스턴스가 소멸되기 직전 `deinit` 메서드가 호출된다.    
클래스 내부에 `deinit` 메서드를 구현해주면 소멸되기 직전 `deinit` 메서드가 호출되며, 이렇게 호출되는 메서드를 __디이니셜라이저(Deinitializer)__ 라고 부른다.

`deinit`메서드는 클래스당 하나만 구현 가능하며, 매개변수와 반환 값을 가질 수 없고 매개변수를 기입하는 소괄호도 적지 않는다.

~~~swift
class Person {
    var heigth: Float = 0.0
    var weight: Float = 0.0
    
    deinit {
        print("perform the deinitialization")
    }
}

var faker: Person? = Person()
faker = nil // perform the deinitialization
~~~


## Comparing Structures and Classes
Swift에서 구조체와 클래스는 비슷한 점이 많다. 
### Same thing
* 값을 저장하기 위해 프로퍼티를 정의할 수 있다.
* 기능 실행을 위해 메서드를 정의할 수 있다.
* 서프스크립트 문법을 통해 구조체 또는 클래스가 갖는 프로퍼티에 접근하도록 정의할 수 있다.
* 초기화될 때의 상태를 지정하기 위해 이니셜라이저를 정의할 수 있다.
* 초기구현과 기능 추가를 위해 익스텐션을 통해 확장할 수 있다.
* 특정 기능을 실행하기 위해 특정 프로토콜을 준수할 수 있다.
### Difference
* 구조체는 상속할 수 없다.
* 디이니셜라이저는 클래스의 인스턴스에만 활용할 수 있다.
* 참조 회수 계간(Reference Counting)은 클래스 인스턴스에만 적용된다.
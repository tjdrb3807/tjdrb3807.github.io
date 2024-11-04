---
layout: post
title: "Swift #Section04 - Closure"
sitemap: false
image: /assets/img/blog/thumbnail/swift/swift_section04_closure.png
categories: [study, swift]
tags: [swift]
description: 본 글에서는 클로저의 다양한 형태 및 표현법과 탈출, 자동 클로저에 대해 다룬다.
---

* toc
{:toc}

## 🤚 Introduce
***클로저는 일정 기능을 하는 코드를 하나의 블록에 모아놓은 것을 말하다.*** 이는 함수의 정의와 매우 유사하며, 함수는 클로저의 한 형태이다. 클로저는 변수나 상수가 선언된 위치에서 **참조(Reference)**를 **획득(Capture)**하고 저장할 수 있다. 

### 클로저의 세 가지 형태
클로저는 세 가지 형태로 존재하며 이 중 하나가 함수이다. 

1. 이름이 있으며, 어떤 값도 획득하지 않는 전역함수 형태
2. 이름이 있으며, 다른 함수 내부의 값을 획득할 수 있는 중접 함수 형태
3. 이름이 없으며, 주변 컨텍스트에 따라 값을 획득할 수 있는 축약 문법으로 작성된 형태

### 다양한 클로저 표현 방법
클로저 표현 방법은 클로저가 함수의 모습이 아닌 하나의 블록의 모습으로 표현될 수 있는 방법을 의미한다. 클로저 위치를 기준으로 크게 기본 클로저 표현과 후행 클로저 표현이 있으며, 각 표현 내에서 가독성을 해치지 않는 선으로 표현을 생략하거나 축약할 수 있다. 
* 매개변수와 반환 값의 타입을 컨텍스트를 통해 유추할 수 있으므로 매개변수와 반환 타입을 생략할 수 있다.
* 블록 내부에 단 한 줄의 표현만 들어있다면 암시적으로 이를 반환 값으로 취급한다.
* 축약된 전달인자 이름을 사용할 수 있다.
* 후행 클로저 문법을 적용할 수 있다.

### sorted(by:)
본 글에서 클로저를 설명하기 앞서 모든 예제 코드에서 사용될 `Array.sorted(by:)` 메서드에 대해 소개한다. 해당 메서드를 사용해 동일한 기능을 하는 코드를 어떻게 간결하게 표현하는지 비교해본다.

`sorted(by:)` 메서드는 클로저를 통해 배열의 요소를 어떻게 정렬할 것인가에 대한 정보를 받아 처리하고 입력받은 배열의 타입과 크기가 동일한 배열을 결과값으로 반환한다.

~~~swift
// file: "Array.sorted(by:)"
@inlinable public mutating func sort(by areInIncreasingOrder: (Element, Element) throws -> Bool) rethrows -> [Element]
~~~

`sorted(by:)` 메서드의 `areInIncreasingOrder` 전달인자에 전달할 `String` 타입 두 개를 가지며, `Bool` 타입을 반환하는 함수를 구현해 결과값을 확인한다.

~~~swift
// file: "Code01.swift"
let names = ["Zeus", "Owner", "Faker", "Gumayushi", "Keria"]

func backwards(first: String, second: String) -> Bool { return first > second }

let reversed = names.sorted(by: backwards)
print(reversed) // ["Zeus", "Owner", "Keria", "Gumayushi", "Faker"]
~~~

## 기본 클로저
클로저의 표현은 통상 다음 형식을 따른다.
~~~swift
// file: "Closure definition"
{ (parameters..) -> return_Type in 
    // statements..
}
~~~

Code01는 함수 이름부터 매개변수 표현등 `first > second`라는 반환 값을 받기 위해 너무 많은 표현이 사용되는 것을 확인할 수 있다. 클로저를 사용해 Code01이 경량된 작업을 확인한다.

~~~swift
// file: "Code02.swift"
let reversed = names.sorted(by:{ (first: String, second: String) -> Bool in
    return first > second
})

print(reversed) // ["Zeus", "Owner", "Keria", "Gumayushi", "Faker"]
~~~

`backwards(first:second:)` 함수를 정의해 `sorted(by:)` 메서드의 매개변수로 전달한 Code01에 비해 `sorted(by:)` 메서드의 매개변수에 클로저를 직접 구현한 Code02가 훨씬 간결해지고 직관적으로 변한 것을 확인할 수 있다.

Code02처럼 매개변수에 기본 클로저를 직접 구현하게되면 Code01처럼 메서드로 전달되는 `backward(first:second:)` 함수가 어디에 위치했는지, 어떻게 구현되어 있는지 찾아야하는 번거러움도 덜 수 있다.(코드의 재사용성을 고려해야 하는 상황이라면 Code01처럼 별도의 함수를 구현하는 것을 권장)

## 후행 클로저
**후행 클로저(Trailing Closure)**는 함수나 메서드의 마지막 전달인자로 위치하는 클로저를 함수나 메서드의 소괄호를 닫은 후 작성하는 형태이다. 클로저의 길이가 길어지거나 가독성이 떨어진다 싶으면 후행 클로저(Trailing Closure)를 사용한다.

***후행 클로저는 함수의 맨 마지막 전달인자로 전달되는 클로저에만 해당***되며, `sorted(by:)` 메서드처럼 ***단 하나의 클로저만 전달인자로 전달하는 경우 소괄호를 생략***할 수 있다.

~~~swift
// file: "Code03.swift"
// 후행 클로저 형태
let reversed = names.sorted() { (first: String, second: String) -> Bool in
    return first > second
}

// sroted(by:) 메서드의 소괄호 생략
let reversed = names.sorted { (first: String, second: String) -> Bool in
    return first > second
}
~~~

또한, 매개변수에 클로저가 여러 개 존재할 경우 **다중 후행 클로저** 문법을 사용할 수 있다. 이는 중괄호(`{ }`)를 열고 닫음으로써 클로저를 표현하고 ***첫 번째 클로저의 전달인자 레이블은 생략한다.***

~~~swift
// file: "Code04.swift"
func doSomething(do: (String) -> Void,
                 onSuccess: (Any) -> Void,
                 onFailure: (Error) -> Void) {
    // statements..
}

// 다중 후행 클로저 형태
doSomething { (someString: String) in
    // do closure..
} onSuccess: { (result: Any) in 
    // success closure..
} ofFailure: { (error: Error) in
    // failure closure..
}
~~~

## 클로저 표현 간소화
클로저 각 표현 내에서 가독성을 해치지 않는 선으로 표현을 생략하거나 축약할 수 있다. 

### 컨텍스트를 이용한 타입 추론
메서드의 전달인자로 전달되는 클로저는 메서드에서 요구하는 형태로 전달해야 한다. 즉, 클로저의 매개변수의 타입이나 개수, 반환 타입 등이 메서드의 매개변수 타입과 일치해야 전달인자로 받아질 수 있다. 

***이는 전달인자로 전달된 클로저는 이미 메서드에서 요구하는 적합한 타입을 준수하고 있어 메서드 내에서 클로저의 매개변수 타입, 개수, 반환 타입을 명시하지 않아도 컨텍스트를 통해 유추할 수 있는 뜻이다.*** 따라서 전달인자로 전달하는 클로저를 구현할 때 매개변수 타입이나 반환 타입을 생략할 수 있다.
~~~swift
// file: "Code05.swift"
// Stirng 반환타입 생략, String 매개변수 타입 생략
let reversed = names.sorted { (first, second) in return first > second }
~~~

### 단축 인자 이름
Code05에서 후행 클로저와 다양한 간소화 표현법을 적용했음에도 여전히 의미없는 클로저의 전달인자 이름(`first`, `second`)이 존재한다. Code05에서 더욱 간소화된 표현을 적용하기 위해 Swift에서는 클로저의 전달인자값을 참조할 수 있는 **단축 인자 이름**를 제공한다.

단축 인자 이름은 첫 번째 전달인자부터 `$0`, `$1`, `$2`, ... 순서로 표현되며, 단축 인자 이름을 사용하게 되면 전달인자 및 반환 타입과 실행코드를 구분하기 위해 사용했던 `in` 키워드를 사용할 필요가 없어진다.
~~~swift
// file: "Code06.swift"
let reversed = names.sorted { return $0 > $1 }
~~~

### 암시적 변환
클로저가 반환 값을 갖는 클로저이고 클로저 내부의 실행문이 단 한 줄이면, 암시적으로 해당 실행문을 반환 값으로 사용할 수 있다.
~~~swift
// file: "Code.07.swift"
let reversed = names.sorted { $0 > $1 }
~~~

### 연산자 메서드
~~~swift
// file: "> operator definition"
public func > <T: Comparable>(lhs: T, rhs: T) -> Bool
~~~
클로저는 매개변수의 타입과 반환 타입이 연산자를 구현한 함수의 모양과 동일하다면 연산자만 명시해도 알아서 연산하고 결과 값을 반환한다. 위 코드는 비교 연산자 `>`의 구현 코드이며, 비교 연산자는 함수라는 것을 확인할 수 있며, 함수는 전달인자로 보내가 충분한 조건을 갖고 있다. 
~~~swift
// file: "Code08.swift"
let reversed = names.sorted(by: >)
~~~
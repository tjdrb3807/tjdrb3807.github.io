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

## ✍🏻 기본 클로저
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

## ✍🏻 후행 클로저
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

## ✍🏻 클로저 표현 간소화
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

## ✍🏻 값 획득
클로저는 자신이 정의된 위치의 주변 컨텍스트를 통해 상수나 변수를 **획득(Capture)**할 수 있다. ***값 획득을 통해 클로저는 주변에 정의한 상수나 변수가 더 이상 존재하지 않더라도 해당 상수나 변수의 값을 자신 내부에서 참조하거나 수정할 수 있다.***

~~~swift
// file: "Code09.swift"
func makeIncrementer(forIncrement amount: Int) -> (() -> Int) {
    var runningTotal = 0
    
    func incrementer() -> Int {
        runningTotal += amount
        
        return runningTotal
    }
    
    return incrementer
}
~~~

Code09에서 `makeIncrementer(forIncrement:)` 함수의 반환 타입은 `() -> Int`로 함수 객체를 반환한다. 반환하는 함수는 매개변수를 받지 않고 반환 타입은 `Int`인 함수로, 호출할 때마다 `Int` 타입의 값을 반환한다.

중첩 함수인 `incrementer()`는 `amount`의 값을 `runningTotal`에 더하여 값을 반환하며, `incrementer()` 함수를 `makeIncrementer(forIncrement:)` 함수 외부에 독립적으로 위치시키면 컴파일 에러가 발생한다.

`incrementer()` 함수 주변에 `runningTotal`과 `amount` 변수가 있다면 `incrementer()` 함수는 두 변수의 참조를 획득할 수 있다. 참조를 획득하면 `runningTotal`과 `amount`는 `makeIncrementer(forIncrement:)` 함수의 실행이 끝나도 사라지지 않고 `incrementer()` 함수가 호출될 때마다 계속해서 사용할 수 있다.

~~~swift
// file: "Code10.swift"
let incrementByTwo = makeIncrementer(forIncrement: 2)

let first = incrementByTwo()    // 2
let second = incrementByTwo()   // 4
let third = incrementByTwo()    // 6
~~~

Code10에서 `makeIncrementer(forIncrement:)` 함수를 사용해 `incrementByTwo`라는 이름의 상수에 `increment()` 함수를 할당했으며, `incrementByTwo`를 호출할 때마다 `runningTotal`은 값이 2씩 증가하는 것을 확인할 수 있다.

`incrementer`를 하나 더 생성해주면, `incrementerByTwo`와는 별개의 다른 참조를 갖는 `runningTotal` 변수 값을 확인할 수 있다. 각각의 `incrementer` 한수는 언제 호출 되더라도 자신만의 `runningTotal` 변수를 갖고 카운트 한다. 이는 각각 자신만의 `runningTotal`의 참조를 미디 획득했기 때문이다.
~~~swift
// file: "Code11.swift"
let incrementByTwo = makeIncrementer(forIncrement: 2)
let incrementByTwo02 = makeIncrementer(forIncrement: 2)
let incrementByTen = makeIncrementer(forIncrement: 10)

let first = incrementByTwo()    // 2
let second = incrementByTwo()   // 4
let third = incrementByTwo()    // 6

let first02 = incrementByTwo02()    // 2
let second02 = incrementByTwo02()   // 4
let thire02 = incrementByTwo02()    // 6

let ten = incrementByTen()      // 10
let twenty = incrementByTen()   // 20
let thirty = incrementByTen()   // 30
~~~

## ✍🏻 탈출 클로저 
함수의 전달인자로 전달한 클로저가 함수 종료 후에 호출될 때 클로저가 함수를 **탈출(Escape)**한다고 표현한다. 클로저를 매개변수로 갖는 함수를 선언할 때 매개변수 이름의 콜론(`:`) 뒤에 `@escaping` 키워드를 기입해 클로저가 탈출하는 것을 허용한다고 명시한다.

`@escaping` 키워드를 별도로 명시하지 않는다면 매개변수로 사용되는 클로저는 기본으로 비탈출 클로저이다. 함수로 전달된 클로저가 함수의 동작이 끝난 후 사용할 필요가 없을 때 비탈출 클로저를 사용한다.

클로저가 함수를 탈출할 수 있는 경우 중 하나는 함수 외부에 정의된 변수나 상수에 저장되어 함수가 종료된 후에 사용할 경우이다. 예를 들어 비동기 작업을 위해 컴플리션 핸들러를 전달인자를 이용해 클로저 형태로 받는 함수들이 많다. 함수가 작업을 종료하고 난 이후에 컴플리션 핸들러, 즉 클로저를 호출하기 때문에 클로저는 함수를 탈출해 있어야 한다. 함수의 전달인자로 전달받은 클로저를 다시 반환할 때도 마찬가지다.

~~~swift
// file: "Code12.swift"
typealias VoidVoidClosure = () -> Void

let firstClosure: VoidVoidClosure = { print("Closure A") }
let secondClosure: VoidVoidClosure = { print("Closure B") }

func returnOneClosure(first: @escaping VoidVoidClosure, second: @escaping VoidVoidClosure, shouldReturnFirstClosure: Bool) -> VoidVoidClosure {
    shouldReturnFirstClosure ? first : second
}

let returnedClosure = returnOneClosure(
    first: firstClosure,
    second: secondClosure,
    shouldReturnFirstClosure: true)

returnedClosure() // Closure A

var closures: [VoidVoidClosure] = []

func appendClosure(closure: @escaping VoidVoidClosure) { closures.append(closure) }
~~~

타입 메서드의 매개변수 클로저에 `@escaping` 키워드를 사용하여 탈출 클로저입을 명시한 경우, 클로저 내부에서 해당 타입의 프로퍼티나 메서드, 서브스크립트 등에 접근할 때 `self` 키워드를 명시적으로 사용해야 한다. (비탈출 클로저는 생략 가능)
~~~swift
// file: "Code13.swift"
typealias VoidVoidClosure = () -> Void

func functionWithNoescapgeClosure(closure: VoidVoidClosure) { closure() }
func functionWithEscapingClosure(completionHandler: @escaping VoidVoidClosure) -> VoidVoidClosure {
    completionHandler
}

class SomeClass {
    var x = 10
    
    func runNoescapeClosure() {
        functionWithNoescapgeClosure {
            x = 200
        }
    }
    
    func runEscapingClosure() -> VoidVoidClosure {
        functionWithEscapingClosure {
            self.x = 100
        }
    }
}

let instance = SomeClass()
instance.runNoescapeClosure()
print(instance.x)   // 200

let returnedClosure = instance.runEscapingClosure()
returnedClosure()
print(instance.x)   // 100
~~~

### withoutActuallyEscaping
비탈출 클로저로 전달인자로 전달한 클로저가 탈출 클로저인 척 해야 하는 경우가 발생한다. 실제로는 탈출하지 않는데 다른 함수에서 탈출 클로저를 요구하는 상황에 해당한다.
~~~swift
// file: "Code14.swift:
func hasElements(in array: [Int], match predicate: (Int) -> Bool) -> Bool {
    array.lazy.filter { predicate($0) }.isEmpty == false // 컴파일 에러!
}
~~~
Code14에서 `hasElements(in:match:)` 함수는 `@escaping` 키워드가 없으므로 비탈출 클로저를 전달받게 된다. 내부에서는 `lazy` 컬렉션에 있는 `filter` 메서드의 매개변수로 비탈출 클로저를 전달한다. 이때 `lazy` 컬렉션은 비동기 작업을 할 때 사용하므로 `filter`메서드가 요구하는 클로저는 탈출 클로저이다.

하지만 `hasElements(in:match:)` 함수 전체를 보면 `match` 매개변수로 전달되는 클로저가 탈출할 필요가 없다. 이때 해당 클로저를 탈출 클로저 처럼 사용할 수 있게 돋는 `withoutActuallyEscaping(_:do:)` 함수가 존재한다.

`withoutActuallyEscaping(_:do:)` 함수의 첫 번째 전달인자로는 탈출 클로저인 척해야 하는 클로저가 전달된다. `do` 전달인자는 비탈출 클로저를 또 매개변수로 전달받아 실제로 작업을 실행할 탈출 클로저를 전달한다.
~~~swift
// file: "Code15.swift"
let numbers = [2, 4, 6, 8]

let evenNumberPredicate = { (number: Int) -> Bool in
    number % 2 == 0
}

let oddNumberPredicate = { (number: Int) -> Bool in
    number % 2 == 1
}

func hasElements(in array: [Int], match predicate: (Int) -> Bool) -> Bool {
    withoutActuallyEscaping(
        predicate,
        do: { escapablePredicate in
            array.lazy.filter { escapablePredicate($0) }.isEmpty == false
        })
}

let hasEvenNumber = hasElements(in: numbers, match: evenNumberPredicate)
let hasOddNumber = hasElements(in: numbers, match: oddNumberPredicate)

print(hasEvenNumber)    // true
print(hasOddNumber)     // false
~~~


## ✍🏻 자동 클로저
함수의 전달인자로 전달하는 표현을 자동으로 변환해주는 클로저를 **자동 클로저(Auto Closure)**라고 하며, ***자동 클로저는 전달인자를 갖지 않는다.*** 자동 클로저는 호출되었을 때 자신이 감싸고 있는 코드의 결과값을 반환한다. 

자동 클로저는 클로저가 호출되기 전까지 클로저 내부의 코드가 동작하지 않는다. 따라서 연산을 지연시킬 수 있다. 이 과정은 연산에 자원을 많이 소모하거나 부작용이 우려될 때 유용하게 사용할 수 있다.
~~~swift
// file: "Code16.swift"
var customersInLine = ["Zeus", "Owner", "Faker", "Gumaushi", "Keria"]
print(customersInLine.count)    // 5

let customerProvider: () -> String = {
    customersInLine.removeFirst()
}

print(customersInLine.count)    // 5

print("Now serving \(customerProvider())!") // Now serving Zeus!
print(customersInLine.count)    // 4
~~~

---

함수의 매개변수로 전달하는 표현을 자동으로 래핑해주는 클로저를 **자동 클로저(Auto Closure)**라 칭한다. 이때 표현식을 자동으로 래핑해준다의 의미는 ***전달인자로 넘어오는 표현식이 클로저 표현이 아니더라도 클로저로 만들어 준다**는 의미이다.

자동 클로저는 전달인자를 갖지 않으며, 호출되었을 때 래핑 되어있는 표현식의 결과값을 반환한다.

### 클로저를 이용한 지연 연산
자동 클로저는 클로저가 호출되기 전까지 내부 코드를 실행시키지 않는다. 따라서 연산을 지연시킬 수 있다. 이 과정은 연산에 자원을 많이 소모하거나 사이드 이펙트 발생 우려가 있을 때 유용하게 사용할 수 있다.

~~~swift
// file: "Code17.swift"
var customersInLine = ["Zeus", "Owner", "Faker", "Gumaushi", "Keria"]
print(customersInLine.count)    // 5

let customerProvider = { customersInLine.removeLast() }

let customerProvider02 = customersInLine.removeLast()

print(customerProvider02)       // Keria
print(customersInLine.count)    // 4

print("Now serving \(customerProvider())!") // Now serving Gumaushi!
print(customersInLine.count)    // 3
~~~

함수의 전달인자로 Code16와 같은 조건의 클로저를 전달할 때도 동일한 동작을 기대할 수 있다. 클로저를 파라미터로 전달할 때, 클로저는 실행되지 않으므로 안의 표현식이 연산되지 않고, 함수 내부에서 호출할 때 실행된다.
~~~swift
// file: "Code18.swift"
// 명시적 클로저를 파라미터로 받는 함수
func serve01(_ customerProvider: () -> String) { print("Now serving \(customerProvider())!") }

serve01({customersInLine.removeLast() })    // Now serving Faker!

// 오토 클로저를 파라미터로 받는 함수
func serve02(_ customerProvider: @autoclosure () -> String) { print("Now serving \(customersInLine.removeLast())!") }

serve02(customersInLine.removeLast())       // Now serving Owner!
~~~

### 자동 클로저의 탈출
기본적으로 `@autoclosure` 키워드 속성은 `@noescape` 키워드 속성을 포함한다. 만약 자동 클로저를 탈출하는 클로저로 사용하고 싶다면 `autoclosure` 키워드 뒤에 `@escaping` 키워드를 덧붙여 사용한다.
~~~swift
// file: "Code19.swift"
func returnProvider(_ customerProvider: @autoclosure @escaping () -> String) -> (() -> String) { customerProvider }

let customerProvider = returnProvider(customersInLine.removeLast())
print("Now serving \(customerProvider())!")   // Now serving Zeus!
~~~

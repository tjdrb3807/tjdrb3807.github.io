---
layout: post
title: "Swift Closure"
sitemap: false
image: /assets/img/blog/thumbnail/swift/swift_instance.png
categories: [study, swift]
tags: [swift]
description: 본 글에서는 Swift 사용자 지정 데이터의 인스턴스 생성 및 소멸에 대해 다룬다.
---

* toc
{:toc}

## 🤚 Introduce
**클로저(Closure)**는 일정 기능을 하는 코드를 하나의 블록으로 모아놓은 것을 말한다. 클로저는 변수나 상수가 서언된 위치에서 참조(Reference)를 획득(Capture)할 수 있으며, 이를 변수나 상수의 클로징(잠금)이라 한다.




정렬 클로저?

인라인 클로저?

정렬 클로저는 메서드의 전달인자로 전달되므로 Swift는 파라미터 타입과 반환되는 값의 타입을 유추할 수 있다.

함수나 메서드에 클로저를 인라인 클로저 표현식으로 전달할 때 항상 파라미터 타입과 반환 타입을 유출할 수 있다. 결과적으로 클로저가 함수 또는 메서드의 전달인자로 사용된 때 완전한 형태로 인라인 클로저를 작성할 필요가 없다.

중첩 함수는 


## 클로저 표현식

`sorted(by:)` 메서드를 통해 동일한 기능을 하는 코드를 어떻게 경량화하는지 확인한다.
~~~swift
// file: sorted(by:).swift
@inlinable public func sorted(by areInIncreasingOrder: (Element, Element) throws -> Bool) rethrows -> [Element]
~~~

`sorted(by:)` 메서드의 전달일자로 반는 클로저(`(Element, Element) throws -> Bool`)가 반환하는 Bool 타입의 값은 첫 번째 전달인자 값이 새로 생성되는 배열에서 두 번째 전달인자 값보다 먼저 배치되어야 하는지에 대한 결과값이다. (`true`)를 반환하면 첫 번째 전달인자가 두 번째 전달인자보다 앞에 위치하게 된다.

<br>

~~~swift
// file: "Code01.swift"
let names = ["Zeus", "Owner", "Faker", "Gumayusi", "Keria"]

func backwards(first: String, second: String) -> Bool {
    return first > second
}

let reversed = names.sorted(by: backwards)

print(reversed) // ["Zeus", "Owner", "Keria", "Gumayusi", "Faker"]
~~~

Code01의 `backwards(first:second:)` 함수를 클로저 표현으로 대체하면 다음과 같다

~~~swift
// file: "Code02.swift"
let reversed = names.sorted(by: { (first: String, second: String) -> Bool in
    return first > second
})

print(reversed)
~~~


## 후행 클로저
함수나 메서드의 마지막 전달인자로 위치하는 클로저는 함수나 메서드의 소괄호를 닫은 후 작성해도 된다. 클로저가 길어 가독성이 떨어진다 싶으면 **후행 클로저(Trailing Closure)** 기능을 사용하면 좋다.

후행 클로저는 맨 마지막 전달인자로 전달되는 클로저에만 해당되므로 전달인자로 클로저 여러 개를 전달할 때는 맨 마지막 클로저만 후행 클로저로 사용할 수 있며, 단 하나의 클로저만 전달인자도 저달하는 메서드의 경우 소괄호를 생략할 수 있다.

매개변수에 클로저가 여러개 있는 경우, 다중 후행 클로저 문법을 사용할 수 있다. 다중 후행 클로저를 사용할 경우, 중괄호를 열고 닫음으로써 클로저를 표현하며, 첫 번째 클로저의 전달인자 레이블은 생략한다.

```swift
// file: "Code03.swift"
// 후행 클로저 사용
let reversed = names.sorted() { (first: String, second: String) -> Bool in
    return first > second
}

// 소괄호 생략
let reversed = names.sorted { (first: String, second: String) -> Bool in 
    return first > second
}

func doSomething(do: (String) -> Void,
                 onSuccess: (Any) -> Void,
                 onFailure: (Error) -> Void) {
    // do somethings ...
}

// 다중 후행 클로저
doSomething { (someString: String) in 
    // do cloure
} onSuccess { (result: Any) in
    // success cloure
} onFailure { (error: Error) in
    // failure closure
}
```

### 문맥을 이용한 타입 유추
매서드의 전달인자로 전달하는 클로저는 메서드에서 요구하는 형태로 전달해야 한다. 즉, 매개변수의 타입이나 개수, 반환 타입 등이 같아야 전달인자로서 전달할 수 있다. 이를 다르게 말하면, 전달인자로 전달할 클로저는 이미 적합한 타입을 준수하고 있다 유추할 수 있는 것이다. 따라서 전달인자로 전달하는 클로저를 구현할 때는 매개변수의 타입이나 반환 값의 타입을 굳이 표현해주지 않고 생량하더라도 문제가 없다.

~~~swift
// file: "Code04.swift"
let reversed = name.sorted { (first, second) in 
    return first > second
}
~~~

### 단축 인자 이름
단축 인자 이름은 첫 번째 전달인자부터 `$0`, `$1`, `$2`, `$3`, ... 순서로 표현한다. 단축 인자 표현을 사용하면 매개변수 및 반환 타입과 실행 코드를 구분하기 위해 사용했던 `in` 키워드를 사용할 필요가 없다.

~~~swift
// file: "Code05.swift"
let reversed = nema.sorted { return $0 > $1 }
~~~

### 암시적 반환 표현
클로저가 반환 값을 갖는 클로저이며 클로저 내부의 실행문이 단 한 줄이라면, 암시적으로 실행문을 반환 값으로 사용할 수 있다.

~~~swift
// file: "Code06.swif"
let reversed = name.sorted { $0 > $1 }
~~~

## 값 획득
클로저는 자신이 정의된 위치의 주변 문맥을 통해 상수나 변수를 획득(Capture)할 수 있다. ***값 획득을 통해 클로저는 주변에 정의한 상수나 변수가 더 이상 존재하지 않도라도 해당 상수나 변수의 값을 자신 내부에서 참조하거나 수정할 수 있다.*** 

클로저를 통해 비동기 콜백(Call-Back)을 작성하는 경우, 현재 상태를 미리 획득해 두지 않으면, 실제로 클로저의 기능을 실행하는 순간 주변 상수나 변수가 이미 메모리에 존재하지 않는 경우가 발생한다.
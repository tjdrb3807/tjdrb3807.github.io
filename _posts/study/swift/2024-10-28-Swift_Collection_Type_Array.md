---
layout: post
title: "Swift #CollectionType - Array"
sitemap: false
image: /assets/img/blog/thumbnail/etc/git_section07.png
categories: [study, swift]
tags: [swift]
description: 본 글에서는 Git이 각각의 Branch를 합치는 두 가지 방법에 대해 다룬다.
---

* toc
{:toc}

## Introduce
배열은 같은 타입의 데이터를 나열한 후 순서대로 저장하는 형태의 컬렉션 타입이다. 값이 중복될 수 있으며, 순서를 보장하는 특징을 지닌다.

## 배열 타입 구문
Swift 배열의 타입은 배열에 저장될 값의 타입을 나타내는 `Array<Element>`로 작성하며, 축약혁으로 `[Element]`로 작성할 수도 있다.

## 빈 배열 생성
초기화 구분을 사용해서 타입을 포함한 빈 배열을 생성할 수 있다.
```Swift
var someInts: [Int] = []
print("someInts of type [Int] with \(someInts.count) items.")   
// someInts of type [Int] with 0 items.
```

## 기본값 배열 생성
Swift의 `Array` 타입은 같은 기본 값으로 설정하고 크기를 고정하여 배열을 생성하는 초기화도 제공한다. 적합한 타입의 기본값과 새로운 배열에 반복될 값의 횟수를 초기화에 전달한다.
Array 구조체의 사용자 지정 이니셜라이저를 통해 repeating 파라미터에는 적합한 타입의 기본값, count 파라미터에는 반복 횟수를 전다랗여 초기화 한다.

```Swift
var threeDoubles = Array(repeating: 0.0, count: 3)
print("threeDoubles is of type [Double], and equals \(threeDoubles)")
//threeDoubles is of type [Double], and equals [0.0, 0.0, 0.0]
```

## 배열을 더해 생성
동등한 타입의 2개의 배열을 `+`(덧셈 연산자)를 통해 합쳐서 새로운 배열을 생성할 수 있다. 새로운 배열의 타입은 합쳐진 2개의 배열의 타입으로부터 추론된다.
```Swift
var anotherThreeDoubles = Array(repeating: 2.5, count: 3)
print("anotherThreeDoubles is of type [Double], and equals \(anotherThreeDoubles)")
// anotherThreeDoubles is of type [Double], and equals [2.5, 2.5, 2.5]

var sixDoubles = threeDoubles + anotherThreeDoubles
print("sixDoubles is inferred as [Double], and equals \(sixDoubles)")
// sixDoubles is inferred as [Double], and equals [0.0, 0.0, 0.0, 2.5, 2.5, 2.5]
```

## 배열 리터럴로 생성
배열 콜렉션으로 하나 이상의 값을 작성하여 배열 리터럴(array literal)로 배열을 생성할 수 있다. 배열 리터럴은 값을 리스트로 작성하고 ,(콤마)로 구분하며 대괄호로 둘러싸서 작성한다.
```Swift
var shoppingList: [String] = ["Eggs", "Milk"]
```
`shoppingList` 변수는 `[String]`으로 쓰고 "분자열 값의 배열"로 선언된다. 이 배열은 `String`의 값 타입을 가지고 있기 때문에 `String` 값만 저장이 가능하다. 여기서 `shoppingList` 배열은 리터럴 안에 쓰여진 2개의 `String` 값("Eggs"와 "Milk")으로 초기화 되었다. 
이 경우 배열 리터럴은 2개의 `String` 값을 포함한다. 이것은 `shoppingList` 변수의 선언 타입(`String` 값만 포함될 수 있는 배열)과 일치하므로 2개의 초기 항목으로 shoppingList를 초기화하는 방법으로 배열 리터럴의 할당이 허용된다.

## 배열의 접근과 수정
Array의 인스턴스 메서드나 프로퍼티 또는 서브 스크립트 구문을 사용하여 배열에 접근 및 수정이 가능하다. 

### Array.count
배열의 아이템 개수를 확인하기 위해서는 Array 인스턴스의 읽기 전용 연산 프로퍼티 `count`로 확인할 수 있다.
```Swift
print("The shopping list contains \(shoppingList.count) items.")
// The shopping list contains 2 items.
```

### Array.isEmpty
Array 인스턴스의 연산 프로퍼티 `isEmpty`를 통해 배열의 `count` 프로퍼티의 값이 0인지 아닌지 확인할 수 있며, 해당 프로퍼티는 `Bool` 타입이다.
```Swift
if shoppingList.isEmpty {
    print("My shopping list is empty.")
} else {
    print("The shopping list is not empty.")
}
//The shopping list is not empty.
```

### Array.append(_:)
`append(_:)` 메서드를 사용하여 배열의 마지막 인덱스에 새로운 아이템을 추가할 수 있다.
```Swift
shoppingList.append("Flour")
print("shoppingList now contains \(shoppingList.count) items.")
// shoppingList now contains 3 items.
```

또한 하나 이상의 같은 타입의 배열은 덧셈 연산자 (`+=`)를 통해 추가할 수 있다.
```Swift
shoppingList += ["Baking Powder"]
print("shoppingList now contains \(shoppingList.count) items.")
// shoppingList now contains 4 items.

shoppingList += ["Chocolate Spread", "cheese", "Butter"]
print("shoppingList now contains \(shoppingList.count) items.")
// shoppingList now contains 7 items.
```

### Subscript
서브스크립트를 사용하여 배열의 값을 가져올 수 있다. 배열의 이름 뒤에 대괄호(`[]`)를 붙이고 가져올 값의 인덱스를 넣어 해당 값을 가져올 수 있다.
```Swift
var firstItem = shoppingList[0]
print("firstItem is equal to \(firstItem)")
// firstItem is equal to Eggs
```

서브 스크립트 구문을 사용하여 존재하는 값을 변경할 수 있다.
```Swift
shoppingList[0] = "Six eggs"
print("the first item in the list is now equal to \(shoppingList[0]) rather than \(firstItem)")
// the first item in the list is now equal to Six eggs rather than Eggs
```

서브 스크립트 구문을 사용할 때 인덱스는 유효해야 한다. 만약 `shoppingList[shoppingList.count] = "Salt"`으로 배열 끝에 추가하려고 하면 런타임 에러가 발생한다.
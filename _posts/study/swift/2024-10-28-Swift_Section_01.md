---
layout: post
title: "Swift #Section01 - Data Type"
sitemap: false
image: /assets/img/blog/thumbnail/etc/git_section07.png
categories: [study, swift]
tags: [swift]
description: 본 글에서는 Git이 각각의 Branch를 합치는 두 가지 방법에 대해 다룬다.
---

* toc
{:toc}

## Intro
데이터 타입은 프로그램 내에서 다워지는 데이터의 종류를 뜻한다.    
Swift의 기본 데이터 타입은 모두 구조체(Struct)를 기반으로 구현되어 있다.

## Int와 UInt
정수(Integers)는 소수가 아닌 전체 숫자를 의미하며, 정수는 부호가 있는 정수(Signed)와 부호가 없는 정수(UnSigned)로 구분된다.   
* `Int`: '+', '-'와 같이 부호를 포함하는 정수
* `UInt`: '-' 부호를 포함하지 않으며, 0을 포함하는 정수

`Int`와 `UInt`타입의 최대값과 최소값은 `max`, `min` 프로퍼티를 통해 확인할 수 있다.

`Int`와 `UInt`는 각각 8bit, 16bit, 32bit, 64bit의 형태가 존재하며, `Int8`, `Int16`, `Int32`, `Int64`, `UInt8`, `UInt16`, `UInt32`, `UInt64`의 타입으로 구분된다.    
사용자의 시스템 아키텍처에 따라 Int와 UInt가 지정된다.

```Swift
print("Max value of Int Type: \(Int.max)")
print("Min value of Int Type: \(Int.min)")

print("Max value of Int32 Type: \(Int32.max)")
print("Min value of UInt32 Type: \(UInt32.min)")
```

해당 PC는 Int타입이 Int64 타입으로 지정된 것을 확인할 수 있다. 

## Bool
Booleans 타입은 참(true) 또는 거짓(false)만 값으로 갖는다. 

## Float와 Double
`Float`와 `Double`은 부동소수점을 사용하는 부동소수 타입(Floating Point Type)이라 한다. 소수점 자리가 있는 수를 의미하며, 부동 소수 타입은 정수 타입보다 훨씬 더 넓은 범위의 수를 표현한 수 있다.    
Swift에서는 64bit의 부동 소수를 표현하는 `Double`과 32bit의 부동 소수를 표현하는 `Float`가 존재한다.

64bit 환경에서 `Double`은 최소 15자리의 십진수를 표현할 수 있으며, `Float`는 6자리의 숫자까지 표현이 가능하다.

> 부동소수값이 10진수로 표현할 수 있는 범위를 넘어서면 콘솔 로그에 출력했을 때 ...

## Character
'문자'를 의미하는 `Character` 타입은 단어  
`Character`를 사용하기 위해서는 `""`를 사용하여 표현할 수 있다.

## String
`String`은 문자의 나열, 즉 문자열 표현하는 타입이다. `Character`와 마찬가지로 `""`를 사용하여 표현할 수 있다.
```Swift
let name: String = "Faker"

// 이니셜라이저를 사용해서 빈 문자열을 생성할 수 있다.
var introduce: String = String()
// ""(큰따옴표)를 사용해서 빈 문자열을 생성할 수 있다.
var phoneNumber: String = ""

// append() 메서드를 활용해서 문자열을 이어붙일 수 있다.
introduce.append("GOAT")

// '+' 연산자를 활용해서 문자열을 이어붙일 수 있다.
phoneNumber += "KR+82" + " " + "010-0000-0000"
```

### String타입이 지원하는 다양한 메서드
## Any, AnyObject와 nil
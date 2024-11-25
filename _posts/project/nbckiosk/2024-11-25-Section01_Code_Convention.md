---
layout: post
title: "#Secion01 - Code Convention"
sitemap: false
# image: /assets/img/blog/thumbnail/etc/git_section07.png
categories: [project, nbckiosk]
tags: [nbckiosk]
# description: App이 Foreground 또는 Background 상태에 있을 때 시스템 Notification에 응답하고 기타 중요 시스템관련 이벤트를 처리한다.
---

* toc
{:toc}

## Introduce
소프트웨어 개발의 80%는 유지보수에 집중된다. 유지보수 과정에서 가독성 높은 코드는 코드 리뷰 및 협업에 유용하다.     
위와같은 목적을 위해 본 프로젝트는 Swift API Design Guidelines을 참고한 다음 코드 컨벤션을 준수한다.

## 네이밍
클래스(타입, 프로토콜 이름 포함) 이름에는 ***UpperCamelCase***(첫 문자를 대문자로 시작하는 camel 표기법), 함수 이름에는 ***camelCase를*** 사용한다.

### 변수, 상수
일반 변수 / 상수인 경우 별도의 접두사를 붙이지 않는다.
~~~swift
let maximumNumberOfLines = 3        // ✅ Preferred

let kMaximumNumberOfLines = 3       // ⛔️ Not Preferred
let MAX_LINES = 3                   // ⛔️ Not Preferred
~~~

### static
전역 변수 / 상수인 경우 앞에 ***k***를 붙여준다.
~~~swift
static let kMaximumNumberOfLines = 3    // ✅ Preferred

let kMaximumNumberOfLines = 3           // ⛔️ Not Preferred
~~~

### 함수01
함수, 메서드 이름에는 되도록 ***get***을 붙이지 않는다.
~~~swift
func name(for user: User) -> String?        // ✅ Preferred

func getName(for: user: User) -> String?    // ⛔️ Not Preferred
~~~

### 함수02
액션 함수의 네이밍은 ***"주어 + 동사 + 목적어"*** 형태를 사용한다.       
***will***은 특정 행위가 일어나기 진전이며, ***did***는 특정 행위가 일어난 직후이다.
~~~swift
func backButtonDidTap() { }     // ✅ Preferred

func back() { }                 // ⛔️ Not Preferred
func pressBack() { }            // ⛔️ Not Preferred
~~~

### 약어
약어로 시작하는 경우 소문자로 표기하고, 그 외 경우에는 항상 대문자로 표기한다.
~~~swift
let userID: Int?         // ✅ Preferred
let html: String?        // ✅ Preferred
let websiteURL: URL?     // ✅ Preferred
let urlString: String?   // ✅ Preferred

let userId: Int?         // ⛔️ Not Preferred
let HTML: String?        // ⛔️ Not Preferred
let websiteUrl: NSURL?   // ⛔️ Not Preferred
let URLString: String?   // ⛔️ Not Preferred
~~~

## 타입
`Array<T>`와 `Dictionaray<T: U>` 보다는 `[T]`, `[T: U]`를 사용한다.
~~~swift
var message: [String]?       // ✅ Preferred
var names: [Int: String]     // ✅ Preferred 

var message: Array<String>?             // ⛔️ Not Preferred
var names: Dictionary<Int, String>?     // ⛔️ Not Preferred
~~~

## self
문법의 모호함을 제거하기 위해 언어에서 필수로 요구하지 않는 이상 `self` 프로퍼티는 사용하지 않는다.
~~~swift
final class Listing {
	private let isFimilyFriendly: Bool
	private var capacity: Int
	
  init(capacity: Int, allowsPets: Bool) {
    self.capacity = capacity            // ✅ Preferred 
    isFamilyFriendly = !allowsPets
 
    self.capacity = capacity
    self.isFamilyFriendly = !allowsPets // ⛔️ Not Preferred
  }
}
~~~

## 패턴
프로퍼티의 초기화는 가능하면 init에서하고 가능하면 Unwrapped Optional의 사용을 지양한다.
~~~swift
class SomeClass: NSObject {   // ✅ Preferred
  var someValue: Int
  
	init() {
		someValue = 0
		super.init()
	}
}

class MyClass: NSObject {     // ⛔️ Not Preferred
	var someValue: Int?
	
  init() {
    super.init()
  }
}
~~~

## return 
return은 생략하지 않는다.
~~~swift
// ✅ Preferred
["1", "2", "3"].compactMap { return Int($0) }

var size: CGSize {
  return CGSize(
    width: 100.0,
    height: 100.0)
}

func makeInfoAlert(message: String) -> UIAlertController {
  return UIAlertController(
    title: "ℹ️ Info",
    message: message,
    preferredStyle: .alert)
}

// ⛔️ Not Preferred
["1", "2", "3"].compactMap { Int($0) }

var size: CGSize {
  CGSize(
    width: 100.0,
    height: 100.0)
}

func makeInfoAlert(message: String) -> UIAlertController {
  UIAlertController(
    title: "ℹ️ Info",
    message: message,
    preferredStyle: .alert)
}
~~~
---
layout: post
title: "2025-01-15 TIL"
sitemap: false
categories: [study, til]
tags: [til]
---

* toc
{:toc}

## Method Swizzling?

"Swizzle: 뒤섞다"

Objective-C에 의해 만들어진 swift언어도 역시 Objective-C의 성격을 가지고 있으므로 메소드의 실행을 런타임에 결정한다.

메소드의 실행은 런타임때 결정되므로, 런타임에서 메소드가 실행되기전에 원하는 내용으로 뒤섰기가 가능하다.

= 런타임 시점에 내가 원하는 코드가 실행되게끔 가능

특정 SDK의 메소드가 있을 때, 이 메소드를 변경하고 싶은 경우 매우 유용하게 사용(초기 SDK의 메소드를 수정할 수 없을 때)

## Method Swizzling 구현 방법

원본 메서드가 @objc dynamic 메서드여야 가능하다.

~~~swift
import UIKit

class ViewController: UIViewController {
    @objc dynamic private func someMethod() {
        print("CALL: Original method.")
    }
}
~~~

`soemMethod()` 대신에 다른 메서드를 실행하고 싶은 경우?

`clasee_getInstanceMethod`라는 함수를 통해 메ㅓ드 인스턴스를 획득

`method_exchangeImplemtations(_:_:)`함수를 통해 런타임 때 메서드 변경

~~~swift
extension ViewController {
    class func swizzleMethod() {
        guard let originalMethod = class_getInstanceMethod(Self.self, #selector(Self.someMethod)),
              let swizzledMethod = class_getInstanceMethod(Self.self, #selector(Self.anotherMethod)) else { return }
            
        method_exchangeImplementations(originalMethod, swizzledMethod)
    }
}

@objc private func anotherMethod() {
    print("CALL: Swizzled method.")
}
~~~

swizzleMethod 실행 후 이전의 someMethod를 실행하면, anotherMethod가 실행된다.

~~~swift
class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()

        self.swizzleMethod()
        self.someMethod()   // CLL: Swizzeld method
    }

    @objc dynamic private func someMethod() {
        print("CALL: someMethod")
    }
}
~~~
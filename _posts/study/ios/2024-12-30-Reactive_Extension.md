---
layout: post
title: "iOS - Reactive Extension"
sitemap: false
categories: [study, ios]
tags: [ios]
---

* toc
{:toc}

## Introduce
Reactive Extension은 UIKit 컴포넌트와 RxSwift의 데이터 스트림을 연결하기 위한 방법이다.   
이를 통해 기존 클래스의 동작을 Reactive 스타일로 확장하거나, 새로운 기능을 추가할 수 있다.   

RxSwift와 RxCocoa에서는 Reactive Extension을 사용해서 UIKit의 기본 컴포넌트를 Reactive 방식으로 확장하고, 이를 통해 Observable, Observer, Binder 등을 활용한 선언적 프로그래밍이 가능하다.


## Reactive Extension의 기본 구조
~~~swift
extension Reactive where Base: SomeClass {
    var someProperty: Observable<String> {
        return Observable.create { observer in
            // Observable 생성 및 데이터 방출
        }
    }
}
~~~

* Reactive: RxSwift에서 제공하는 제네릭 구조체로, Reactive Extension의 기반이 된다.
* Base: Reactive Extension이 적용되는 원래 클래스의 타입(ex: UILabel, UIViewController...)
* where Base: SomeClass: 확장이 특정 클래스에만 적용되도록 제약을 추가

## Reactive Extension 매커니즘
### RxSwift의 Reactive 구조체
* Reactive 구조체는 특정 클래스를 Reactive 방식으로 확장하는데 사용된다. 이를 통해 원래 클래스가 지원하지 않는 Reactive Extension(Observable, Binder 등)을 추가할 수 있다.
* 모든 Reactive Extension은 해당 구조체를 기반으로 작성된다.

~~~swift
@dynamicMemberLookup
public struct Reactive<Base> {
    /// Base object to extend.
    public let base: Base

    /// Creates extensions with base object.
    ///
    /// - parameter base: Base object.
    public init(_ base: Base) {
        self.base = base
    }

    /// Automatically synthesized binder for a key path between the reactive
    /// base and one of its properties
    public subscript<Property>(dynamicMember keyPath: ReferenceWritableKeyPath<Base, Property>) -> Binder<Property> where Base: AnyObject {
        Binder(self.base) { base, value in
            base[keyPath: keyPath] = value
        }
    }
}
~~~

* Base
  * Reactive 구조체는 특정 클래스 또는 객체를 Reactive 방식으로 확장하기 위해 해당 객체를 래핑한다.
  * base 프로퍼티는 래핑된 원래 객체를 참조한다.
* 제네릭 타입
  * Reactive<Base>는 제네릭 구조체이므로, 어떤 타입이든 Reactive 방식으로 확장할 수 있다.

### ReactiveCompatible 프로토콜
* Reactive Extension을 사용하려면 대상 클래스가 ReactiveCompatible 프로토콜을 채택해야 한다.
* UIKit 클래스들은 RxCocoa에서 기본적으로 ReactiveCompatible을 채택하고 있다.

~~~swift
/// A type that has reactive extensions.
public protocol ReactiveCompatible {
    /// Extended type
    associatedtype ReactiveBase

    /// Reactive extensions.
    static var rx: Reactive<ReactiveBase>.Type { get set }

    /// Reactive extensions.
    var rx: Reactive<ReactiveBase> { get set }
}
~~~

### Reactive 속성 추가
* Reactive Extension을 추가하면 대상 클래스에 rx 속성이 자동으로 생긴다.

~~~swift
extension ReactiveCompatible {
    /// Reactive extensions.
    public static var rx: Reactive<Self>.Type {
        get { Reactive<Self>.self }
        // this enables using Reactive to "mutate" base type
        // swiftlint:disable:next unused_setter_value
        set { }
    }

    /// Reactive extensions.
    public var rx: Reactive<Self> {
        get { Reactive(self) }
        // this enables using Reactive to "mutate" base object
        // swiftlint:disable:next unused_setter_value
        set { }
    }
}
~~~


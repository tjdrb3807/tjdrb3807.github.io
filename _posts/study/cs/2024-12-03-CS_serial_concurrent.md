---
layout: post
title: "Computer Science - Serial & Concurrent(01)"
sitemap: false
categories: [study, cs]
tags: [cs]
---

* toc
{:toc}

## Serial Queue
Serial Queue는 한 번에 하나의 작업만 실행할 수 있는 Dispatch Queue의 한 종류이다.     

Serial Queue에 작업이 등록되는 순서대로 실행되며, 다음 작업은 현재 작업이 끝난 후에 실행되기 때문에 작업 순서가 보장된다.

GDC가 제공하는 Main Queue가 대표적인 Serial Queue에 속한다.

### Serial Queue에서 Async 호출
Async를 사용하면 이를 호출한 Thread가 해당 작업이 끝날 때까지 대기하지 않고, 바로 다음 코드를 실행한다. 

하지만 Serial Queue의 특성상 작업은 하나씩 순차적으로 실행되므로 Async로 작업을 추가하더라도 Queue 내부에서는 동기적으로 실행된다.
~~~swift
// Main Thread에서 호출한다 가정
let someSerialQueue = DispatchQueue(label: "someSerialQueue")

someSerialQueue.async {
    print("START: Task01")
    sleep(1)
    print("END: Task02")
}

someSerialQueue.async {
    print("START: Task02")
    sleep(1)
    print("END: Task02")
}

someSerialQueue.async {
    print("START: Task03")
    print("END: Task03")
}

// someSerialQueue <- Main Thread Task01
// someSerialQueue <- Main Thread Task02
// someSerialQueue <- Main Thread Task03

// someSerialQueue[Task01, Task02, Task03]

// START: Task01        1초 텀
// ENE: Task02
// START: Task02        1초 텀
// END: Task02
// START: Task03
// END: Task03
~~~

* Main Thread에서 해당 작업들을 호출했으므로 순차적으로 someSerialQueue에 등록된다.
* async로 작업들을 someSerialQueue에 등록했으므로 이전에 등록한 작업이 종료되는 것을 기다리지 않고 Main Thread는 바로 다음 작업을 someSerialQueue에 등록한다.
* someSerial Queue에 작업1, 2, 3이 등록되었으며, 순차적으로 작업을 실행한다.

## Concurrent Queue
Concurrent Queue는 동시성을 지원하는 Dispatch Queue의 한 종류로, 여러 작업을 동시에 병렬적으로 실행할 수 있다.

이때 작업이 Queue에 추가된 순서대로 실행되기 때문에 시작 순서는 보장되지만, 각 작업의 완료 시간은 작업 내용과 실행 환경에 따라 다르므로 보장되지 않는다.

GDC가 제공하는 Global Queue가 대표적인 Concurrent Queue에 속한다.

~~~swift
// Main Thread에서 호출한다 가정
let someConcurrentQueue = DispatchQueue(
    label: "someConcurrentQueue",
     attributes: .concurrent)

someConcurrentQueue.async {
    print("START: Task01")
    sleep(2)
    print("END: Task01")
}

someConcurrentQueue.async {
    print("START: Task02")
    sleep(2)
    print("END: Task02")
}

someConcurrentQueue.async {
    print("START: Task03")
    print("END: Task03")
}

// someConcurrentQueue <- Main Thread Task01
// someConcurrentQueue <- Main Thread Task02
// someConcurrentQueue <- Main Thread Task03

// someConcurrentQueue[Task01, Task02, Task03]

// 결과가 다를 수 있음
// START: Task01
// START: Task02
// START: Task03
// END: Task03
// END: Task02
// END: Task01
~~~

* Main Thread에서 해당 작업들을 호출했으므로 순차적으로 someConcurrentQueue 등록된다.
* async로 작업들을 someConcurrentQueue 등록했으므로 이전에 등록한 작업이 종료되는 것을 기다리지 않고 Main Thread는 바로 다음 작업을 someConcurrentQueue 등록한다.
* someConcurrentQueue Queue에 작업1, 2, 3이 등록되었으며, 순차적으로 작업을 실행하지만, 이전 작업이 실행중이라도 대기하지 않고 동시에 실행된다.



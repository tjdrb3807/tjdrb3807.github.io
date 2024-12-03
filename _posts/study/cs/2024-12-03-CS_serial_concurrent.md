---
layout: post
title: "Computer Science #Section03 Serial & Concurrent"
sitemap: false
categories: [study, cs]
tags: [cs]
---

## Serial Queue
Serial Queue는 한 번에 하나의 작업만 실행할 수 있는 Dispatch Queue의 한 종류이다.     

Serial Queue에 작업이 등록되는 순서대로 실행되며, 다음 작업은 현재 작업이 끝난 후에 실행되기 때문에 작업 순서가 보장된다.

GDC가 제공하는 Main Queue가 대표적인 Serial Queue에 속한다.

### Serial Queue에 Async 호출
~~~swift
// Main Thread에서 호출한다 가정
let someSerialQueue = DispatchQueue(label: "someSerialQueue")

someSerialQueue.async {
    print("START: Task01")
    sleep(1)
    print("ENE: Task02")
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
* async로 작업들을 호출했으므로 Main Thread는 작업이 실행되는 것을 기다리지 않고 바로 다음 작업을 someSerialQueue에 등록한다.
* someSerial Queue에 작업1, 2, 3이 등록되었으며, 순차적으로 작업을 실행한다.

### Serial Queue에서 sync 호출
~~~swift
// Main Thread에서 호출한다 가정
someSerialQueue.sync {
    print("START: Task01")
    sleep(1)
    print("END: Task01")
}

someSerialQueue.async {
    print("START: Task02")
    print("END: Task02")
}

// someSerialQueue <- Main Thread Task01

// someSerialQueue[Task01]

// START: Task01    // 1초 텀
// END: Task01

// someSerialQueue <- Main Thread Task02

// someSerialQueue[Task02]

// START: Task02
// END: Task02
~~~
* Main Thread에서 해당 작업들을 호출했으므로 순차적으로 someSerialQueue에 등록된다.
* Task01이 sync로 호출됐으므로 someSerialQueue에 등록되고 someSerialQueue 내에서 Task01의 작업이 끝날때까지 Main Thread의 Task02는 대기한다.
* Task01의 작업이 끝난 후 Main Thread에서 Task02를 someSerialQueue에 등록하고 someSerialQueue 내에서 Task02가 순번이 됐을 때 작업을 수행한다.

## Concurrent Queue




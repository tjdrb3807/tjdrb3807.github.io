---
layout: post
title: "Computer Science #Section01 - Memory Structure "
sitemap: false
image: /assets/img/blog/thumbnail/swift/swift_section04_protocol.png
categories: [study, cs]
tags: [cs]
description: 본 글에서는 Swift의 프로토콜 지향 프로그래밍과 응용을 알아보기 전 프로토콜에 대해 다룬다.
---

* toc
{:toc}

## 🤚 Introduce
프로그램이 실행되기 위해서는 프로그램이 메모리에 로드(Load)되어야 한다.
컴퓨터의 운영체제(OS)는 프로그램의 실행을 위해 다양한 메모리(RAM) 공간을 제공하고 각각의 메모리 공간은 상호작용하며 프로그램 실행에 기여한다.

## 메모리 구조
다양한 프로그래밍 언어를 사용해 코드를 작성하게 되면 실행파일이 생성된다.
이러한 실행파일을 실행시키면(모바일 환경의 경우 사용자가 앱을 실행시키는 것을 의미) 운영체제는 프로그램의 정보들을 읽고 메인 메모리에 공간은 할당해준다.
실행파일이 메모리공간에 로드되면 프로그램의 코드(변수, 함수)를 메모리에서 읽고 쓰면서 동작하게 된다.

![Memory-structure image](/assets/img/blog/cs/memory_structure_image01.png){: width="100%" height="100%" loading="lazy"}
메모리 구조 (RAM)
{:.figcaption}

### Code
메모리의 코드(Code) 영역에는 프로그래머가 작성한 소스코드를 기계어 형태로 저장된다.
컴파일 시점에 해당 영역이 등록되는 코드들이 결정되며, 런타임 시 코드가 변경되지 않기 위해 Read-Only 형태로 저장된다.

### Data
데이터 영역에는 코드에서 작성한 전역변수(static)가 저장된다. 
런타임 시작과 동시에 해당 영역에 메모리가 할당되며, 런타임 종료 시점에 메모리에서 해제된다.
실행 도중 변수 값이 변경될 수 있으므로 Read-Write로 저장된다.

데이터 영역은 초기화된 전역변수가 등록되는 영역(Initialized data)와 초기화되지 않은 변수가 등록되는 영역(Uninitialized data)으로 구분된다. 
그중 초기화되지 않은 변수가 할당되는 영역을 BSS(Block Started by Symbol)이라 칭한다.

### Heap
힙 영역은 사용자에 의해 관리되는 영역이다.
동적으로 할당 할 변수들이 해당 영역에 로드되며 Swift에서 클래스 인스턴스, 클로저 같은 Reference Type의 값이 Heap 영역을 차지하게 된다.
Heap 영역은 대개 낮은 주소에서 높은 주소로 할당(적재)된다.
~~~swift
var callByReferenceType = self  // UIViewController
print("Call by reference memory address: ", separator: "", terminator: "")
printMemoryAddress(&callByReferenceType)
print("Reeference Instance memory address: ", separator: "", terminator: "")
printHeapMemoryAddress(self)     // UIViewController

// Call by reference memory address: 0x000000016d8b35c0
// Reeference Instance memory address: 0x0000000103515940
~~~

![Memory-structure image](/assets/img/blog/cs/memory_structure_image06.png){: width="100%" height="50%" loading="lazy"}

`callbyReferenceType`는 `UIViewController`의 참조 주소가 담겨있는 지역변수로서 프로퍼티의 메모리 주소 `0x000000016d8b35c0`는 스레드0 Stack 영역에 할당된 것을 확인할 수 있다.

![Memory-structure image](/assets/img/blog/cs/memory_structure_image07.png){: width="100%" height="50%" loading="lazy"}
실제 `UIViewController` 인스턴스 주소는 `0x0000000103515940`이며 Heap 영역에 할당된 것을 확인할 수 있다.


### Stack
스택 영역은 함수를 호출 할 때 지역변수, 매개변수들이 저장되는 공간입니다.   

Swift는 Value Type을 Stack 영역에 할당된다.
~~~swift
var valueTpyeProperty = "Value Type"
print("Value type memory address: ", separator: "", terminator: "")
printMemoryAddress(&valueTpyeProperty)

// Value type memory address: 0x000000016ee875e8
~~~

![Memory-structure image](/assets/img/blog/cs/memory_structure_image03.png){: width="100%" height="50%" loading="lazy"}
vmmap PID | grep Stack
{:.figcaption}

확인 결과 `valueTypeProperty`의 메모리 주소는 `0x000000016ee875e8` 이므로 스레드0의 스텍 메모리 범위에 할당된 것을 확인할 수 있다.

또한 Call by Value를 통해 Value Type의 값을 복사한 프로퍼티의 주소를 확인해보면
~~~swift
var valueTpyeProperty = "Value Type"
var callByValueProperty = valueTpyeProperty
print("Value type memory address: ", separator: "", terminator: "")
printMemoryAddress(&valueTpyeProperty)
print("Call by value memory address: ", separator: "", terminator: "")
printMemoryAddress(&callByValueProperty)

// Value type memory address: 0x000000016ba635d8
// Call by value memory address: 0x000000016ba635c8
~~~

![Memory-structure image](/assets/img/blog/cs/memory_structure_image05.png){: width="100%" height="50%" loading="lazy"}

확인 결과 `valueTypeProperty`의 메모리 주소는 `0x000000016ba635d8`, `callByValueProperty`의 메모리 주소는 `0x000000016ba635c8`이며, 두 프로퍼티 모두 스레드0 스텍 영역에 할당된 것을 확인할 수 있다.

또한 먼저 메모리 공간에 할당된 valueTypePropery의 주소가 나중에 메모리 공간에 할당된 callByValueProperty의 주소보다 높은것을 확인할 수 있으며, 이를 통해 Stack 영역은 높은 메모리 주소부터 적재하는 방식임을 확인할 수 있다.

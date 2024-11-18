---
layout: post
title: "[Data Structure] Linked List"
sitemap: false
# image: /assets/img/blog/thumbnail/etc/git_section07.png
categories: [algorithm, data-structure]
tags: [data-structure]
# description: App이 Foreground 또는 Background 상태에 있을 때 시스템 Notification에 응답하고 기타 중요 시스템관련 이벤트를 처리한다.
---

* toc
{:toc}

## Introduce
**LinkedList**는 **Array**처럼 순차적으로 연결된 데이터를 보관하는 방법이 아닌, ***연속되지 않은 메모리에 저장된 데이터를 연결한 자료구조이다.*** 위와같은 특징으로 LinkedList는 원하는 때에 메모리 공간에 데이터를 할당해서 쓸 수 있는 장점이 있다.     

또한 Array는 마지막 index가 아닌 element를 삭제하거나 삽입할 경우 재배치 작업으로 인해 오버헤드가 발생하지만, LinkedList는 떨어진 데이터의 연결만 바꿔주면 되므로 ***중간 element를 삽입, 삭제해도 재배치에 발생하는 오버헤드를 고려하지 않아도 된다.***    

하지만 LinkedList는 Array처럼 index를 통해 데이터에 접근하는 것이 아니므로, 데이터에 접근하려면 첫 번째 데이터부터 원하는 데이터까지(단방향 연결리스트) ***순차적으로 찾아가야하는 매커니즘 때문에 접근 속도가 느리다는 단점이 존재한다.***     
또한 다음 데이터에 대한 연결 정보를 저장하는 ***별도의 데이터 공간이 필요해서 저장 공간의 효율이 높지 않다는 단점도 존재한다.***


## Node
LinkedList를 구성하는 element의 데이터 구조를 통해 LinkedList의 매커니즘을 이해할 수 있다.   
LinkedList의 데이터 구조는 현재 위치의 데이터(data)와 다음에 위치할 데이터의 주소값(next)을 가지고 있는 형식이며, 이렇게 구성된 모양을 노드(Node)라 칭한다.
LinkedList는 Node들이 연결되어 이루어진 자료구조임을 뜻한다.

### 노드 구현
* Node.next 프로퍼티에 다음 Node의 참조가 할당되기 위해 class로 구현    
  * Node 객체를 struct로 구현하려면 Hashable, Equtable 프로토콜을 준수하도록 하고, 메서드에서도 값이 아닌 참조를 하도록 강제해야 함
  * struct는 생성 및 할당 시 새로운 인스턴스를 생성, 즉 매번 새롭게 메모리에 적재되는 Value Type인 반면 Class는 할당을 하더라도 참조가 할당되는 Reference Type 이기 떄문.
* 제네릭타입으로 다양한 형태의 값을 받을 수 있는 var data: T 
* 다음 노드의 참조를 저장할 next 선언 및 다음 Node가 없을 수 있으므로 Optional 선언

~~~swift
// file: "Code01.swift"
class Node<T> {
    var data: T
    var next: Node?

    init(data: T, next: Node? = nil) {
        self.data = data
        self.next = next
    }
}

extension Node: CustomStringConvertible {
    var description: String {
        guard let next = next else { return "\(data)" }

        return "\(data) -> " + String(description: next) + " "
    }
}

let node01 = Node(data: 1)
let node02 = Node(data: 2)
let node03 = Node(data: 3)

node01.next = node02
node02.next = node03

print(node01)   // 1 -> 2 -> 3
~~~

## LinkedList
* head: LinkedList의 가장 첫 번째 Node
* tail: LinkedList의 가장 마지막 번째 Node(next 프로퍼티에 nil이 할당됨)
* head Node의 next 프로퍼티의 값이 nil이라면 해당 LinkedList의 head는 tail이기도 함.

~~~swift
// file: "Code02.swift"
struct LinkedList<T> {
    var head: Node<T>?
    var tail: Node<T>?

    var isEmpty: Bool { head == nil }
}
~~~

## LinkedList - Node 추가
LinkedList는 세가지 방법으로 Node를 추가할 수 있다.
1. push: LinkedList의 가장 앞에 Node 추가
2. append: LinkedList의 가장 뒤에 Node 추가
3. insert(after:): LinkedList의 특정 Node 뒤에 Node 추가

### push
push 메서드는 추가하고자 하는 값을 알려주면 해당 값을 지닌 Node를 만들어 head로 선언해주며, 동시에 이전의 head는 next로 선언해준다.     
이떄 tail이 nil이라면 tail을 head로 지정한다.

~~~swift
// file: "Code03.swift"
struct LinkedList<T> {
    // ...
    
    mutating func push(_ data: T) {
        head = Node(data: data, next: head)
        if tail == nil { tail = head }
    }
}

var list = LinkedList<Int>()

list.push(3)
list.push(2)
list.push(1)

print(list) // 1 -> 2 -> 3
~~~

### append
append 메서드는 추가하고자 하는 값을 알려주면 LinkedList의 가장 마지막에 해당 Node를 만들어 추가해준다.
1. LinkedList가 비어있다면, push와 동일하게 push메서드에 값을 넘겨주고 종료한다.
2. LinkedList가 비어있지 않다면, tail의 next에 전달받은 값으로 Node를 생성하고 next로 지정한다.
3. 이때, tail은 반드시 존재하므로, tail!.next처럼 옵셔널을 강제추출해준다.

~~~swift
// file: "Code04.swift"
struct LinkedList<T> {
    // ...

    mutating func append(_ data: T) {
        guard !isEmpty else { 
            push(data: data)
            
            return 
        }

        tail!.next = Node(data: data)
        tail = tail!.next
    }
}

var list = LinkedList<Int>()
list.append(1)
list.append(2)
list.append(3)

print(list) //  1 -> 2 -> 3
~~~

### find(at:)
Node들은 중복된 값을 가질 수 있고, LinkedList는 Array처럼 index를 통해 특정 Node에 접근할 수 없으므로 특정 Node가 몇 번째 Node인지 찾는 메서드를 구현한다.

find(at:) 메서드는 찾고자하는 Node의 index를 주면 해당 index에 해당되는 Node를 찾는 메서드이다.
1. 현재 Node와 현재 index를 변수로 선언하고, 각각 head와 0을 할당한다.
2. 현재 Node 변수는 계속해서 다음 Node를, 현재 index는 계속해서 1을 더해가며 해당 index를 찾거나, tail 다음(nil)에 도달할 때 까지 index에 해당하는 Node를 찾는다.
3. 원했던 Node를 return 한다.

~~~swift
// file: "Code05.swift"
struct LinkedList<T> {
    // ...

    func find(at index: Int) -> Node<T>? {
        var crtNode = head
        var crtIndex = 0

        while crtNode != nil && crtIndex < index {
            crtNode = crtNode!.next
            crtIndex += 1
        }

        return crtNode
    }
}
~~~

### insert(_:after:)
find(at:) 메서드를 통해 insert(_:after:) 메서드 구현
1. 특정 Node가 LinkedList의 tail 이라면, append에 값을 넘겨주고 종료
2. 특정 Node의 next에 새로운 Node를 만듦과 동시에 새로운 Node의 next는 기존 Node의 next로 설정 
   1. 특정 Node의 next를 내가 준 값의 새로운 Node로 설정
   2. 내가 준 값의 새로운 Node의 next는 특정 Node의 원래 next로 설정

~~~swift
// file: "Code06.swift"

struct LinkedList<T> {
    // ...

    @discardableResult  // 해당 함수의 리턴값이 사용되지 않더라고, Swift 컴파일러가 경고해줄 필요 없다는 의미
    mutating func insert(_ data: T, after node: Node<T>) -> Node<T> {
        guard tail !== node else {
            append(data)

            return tail!
        }

        node.next = Node(data: data, next: node.next)
        
        return node.next!
    }
}

var list = LinkedList<Int>()
list.push(3)
list.push(2)
list.push(1)

print(list) // 1 -> 2 -> 3

var middleNode = list.find(at: 1)!
for _ in 1...4 {
    middleNode = list.insert(-1, after: middleNode)
}

print(list) // 1 -> 2 -> -1 -> -1 -> -1 -> -1 -> 3
~~~

## LinkedList - Node 삭제
1. pop: LinkedList 내 가장 첫 번째 Node 삭제
2. removeLast: LinkedList 내 가장 마지막 Node 삭제
3. remove(at:) 특정 index의 Node 삭제

### pop
pop 메서드는 LinkedList의 head를 삭제하고 리턴하며, head.next를 새로운 head로 설정하는 메서드이다.    
이때 리턴되는 값은 nil일 수 있으므로 (비어있는 LinkedList를 pop 하는 경우) Optional 값으로 선언한다.

~~~swift
// file: "Code07.swift"
struct LinkedList<T> {
    // ...

    mutating func pop() {
        /*
            defer 키워드는 의도적으로 return 바로 전에 실행하도록, 실행 순서를 가장 뒤로 설정한다.
            defer 키워드를 사용하지 않으면, 기존의 값, 즉 삭제되는 값인 head?.data를 담을 변수가 필요하다.
            하지만 defer를 통해서 return 이후에 head = head?.next를 할당하므로 오직 return만을 위해 새로운 변수를 인스턴스할 필요가 없어진다.
        */
        defer {
            head = head?.next
            if isEmpty { tail = nil }
        }

        return head?.data
    }
}

var list = LinkedList<Int>()
list.push(3)
list.push(2)
list.push(1)

print(list) // 1 -> 2 -> 3

let popData = list.pop()
print(list) // 2 -> 3
print(popData)  // 1
~~~

### removeLast
removeLast 메서드는 tail을 삭제하고, tail 앞의 Node를 tail로 만드는 메서드이다.    
Node에는 next만 있고 prev가 없기때문에 어떤 Node가 tail 앞의 Node인지 알 수 없다. 즉, LinkedList를 처음부터 끝까지 돌면서 tail의 앞 Node가 어떤 Node인지 알아내야 한다.
~~~swift
// file: "Code08.swift"
struct LinkedList<T> {
    // ...

    mutating func removeLast() -> T? {
        guard let head = head else { return nil }   // 빈 LinkedList라면 nil 반환
        guard let head.next != nil else { return pop() }    // head == tail이라면 pop과 동일하므로 pop을 리턴

        var prev = head
        var crt = head

        // tail 이전의 Node가 어떤 Node인지 찾기 위한 탐색
        while let next = crt.next {
            prev = crt
            crt = next
        }

        // tail 이전 Node을 찾았다면 Node의 next를 없애고, tail을 해당 Node로 바꾼 . 후이전 tail 을 리턴
        prev.next = nil
        tail = prev

        return crt.data
    }
}

var list = LinkedList<Int>()
list.push(3)
list.push(2)
list.push(1)

print(list)  // 1 -> 2 -> 3

let removedData = list.removeLast()
print(linst) // 1 -> 2
print(removeData)   // 3 

~~~

### remove(at:)
remove(at:) 메서드는 특정 Node의 next를 삭제하는 메서드이다. 이때 next의 next를 특정 Node와 연결해주게 된다.

~~~swift
// file: "Code09.swift"
struct LinkedList<T> {
	// ...

    mutating func remove(after node: Node<T>) -> T? {
    	defer {
        	if node.next === tail { tail = node }
            node.next = node.next?.next // ⭐️
        }

        return node.next?.data
    }    
}

// 출력 예제
var list = LinkedList<Int>()
list.push(3)
list.push(2)
list.push(1)

print(list) // 1 -> 2 -> 3
let removedValue remove(after: node)
print(list) // 1 -> 3
print(removedValue) // 2
~~~
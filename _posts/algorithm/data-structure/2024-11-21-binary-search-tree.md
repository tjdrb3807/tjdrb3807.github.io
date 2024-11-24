---
layout: post
title: "[Data Structure] Binary Search Tree(BST)"
sitemap: false
# image: /assets/img/blog/thumbnail/etc/git_section07.png
categories: [algorithm, data-structure]
tags: [data-structure]
# description: App이 Foreground 또는 Background 상태에 있을 때 시스템 Notification에 응답하고 기타 중요 시스템관련 이벤트를 처리한다.
---

* toc
{:toc}

[**📌 구현 코드**](https://github.com/tjdrb3807/Data_Structure/tree/main/Tree/BinarySearchTree)   

## 이진트리(Binary Tree)
이진트리(Binary Tree)는 데이터를 계층화하는 Tree 자료구조의 한 종류이다.
트리에서 이진트리가 별도로 분류되는 기준은 다음과 같다.
기본적으로 트리(Tree)를 구성하는 Node들은 Child Node 개수에 제약 없다. 반면 이진트리는 최대 2개의 Child Node를 가질 수 있는 제약이 존재한다. 따라서 Edge를 통해 0~2개의 Child Node 들로 구성된 트리를 이진트리로 분류한다.


## 이진탐색트리(Binary Search Tree)
이진탐색트리는 정렬된 이진트리로써 기존 이존트리의 특징에 추가적인 조건이 담긴다.
* 특정 노드의 왼쪽 Sub tree에는 특정 노드의 Key 값보다 작은 Key가 있는 노드만 포함된다.
* 특정 노드의 오른쪽 Sub Tree에는 특정 노드의 Key 값보다 큰 Key가 있는 노드만 포함된다.
* 중복된 Key를 허용하지 않는다.

## Binary Search Tree (with - Linked List)
이진탐색트리를 구현하기 위해서는 삽입, 검색, 삭제 연산을 필요로 한다.

### Node   
* 이진탐색트리를 구성하는 ***Node는 0~2개의 Child Node를 참조할 수 있으므로***, 참조를 할당할 수 있는 프로퍼티를 2개(`leftChile`, `rightChild`) 선언한다.
* Childe Node를 가리키는 참조는 ***추가, 삽입, 삭제 연산을 통해 변경될 수 있으므로 변수로 선언***한다.
* 이진탐색트리를 구성하는 ***Node는 Child Node를 갖지 않을 수 있으므로 Optional Type으로 선언***한다.

~~~swift
// file: "Node.swift"
class Node<T: Comparable> {
    let value: T
    var leftChild: Node<T>?
    var rightChild: Node<T>?

    init(value: T) {
        self.value = value
    }
}
~~~

### RootNode
이진탐색트리에서 왼쪽 서브트리나 오른쪽 서브트리로 분기를 타기위해서는 가장 처음 기준이 되는 Root Node 프로퍼티가 필요하다. 
이진탐색트리를 구현하기 위해서는 기준이 되는 Root Node 프로퍼티가 필요하다. 
* Root Node로 부터 분기를 통해 다양한 Child Node가 계층에 맞게 위치할 수 있다.
* Root Node를 제외한 모든 Node들은 자신을 참조하는 Parent Node를 갖는다.
* 
* 
~~~swift
// file: "BinarySearchTree.rootNode"
class BinarySearchTree<T: Comparable> {
    var rootNode: Node<T>?
}
~~~


### insert(_:)
#### Logic
* Root Node(`root`) 존재 유무(`nil`) 확인
  * 존재하지 않는다. → Argument(`value`)를 기반으로 TreeNode의 인스턴스 생성 후 `root` Property가 생성한 인스턴스의 참조를 기리킬 수 있도록 할당(함수 종료)
  * 존재한다. → 다음 분기점으로 이동
* 현재 검색중인 노드의 위치를 확인할 수 있는 변수(`crtNode`) 선언 및 `root` 프로퍼티의 참조 할당(얕은 복사)
* 반복문
  * 전달인자의 값과 현재 노드의 값을 비교
    * 전달인자의 값이 더 작다
      * 현재 노드의 왼쪽 노드 존재 유무 확인
        * 왼쪽 노드가 존재하지 않는다. → 현재 노드의 왼쪽 자식 노드가 전달인자를 기반으로 생성한 노드 인스턴스의 참조를 가리키도록 설정 → 함수 종료
        * 왼쪽 노드가 존재한다. → 현재 검색죽인 노드가 현재 검색중인 노드의 왼쪽 자식 노드의 참조를 가리킨 수 있도록 설정
    * 전달인자의 값이 더 크다
      * 현재 노드의 오른쪽 자식 노드 존재 유무 확인
        * 오른쪽 자식 노드가 존재하지 않는다. → 현재 노드의 오른쪽 자식 노드가 전달인자를 기반으로 생성한 노드의 인스턴스의 참조를 가리킬 수 있도록 설정 → 함수 종료
        * 오른쪽 자식 노드가 존재한다 → 현재 검색중인 노드가 현재 검색중인 노드의 오른쪽 자식 노드의 참조를 가리킬 수 있도록 설정
    * 전달인자의 값과 혀재 노드의 값이 같다

#### Code
~~~swift
// file: "BinarySearchTree.insert(_:).swift"
func insert(_ value: T) {
    guard let root = self.root else { return self.root = TreeNode(value: T) }
    var crtNode = root

    while true {
        if value < crtNode.value {
            guard let leftNode = crtNode.left else { crtNode.left = TreeNode(value: T) }
            crtNode = crtNode.left
        } else if value > crtNode.value {
            guard let rightNode = crtNode.right else { crtNode.right = TreeNode(value: T) }
            crtNode = crtNode.right
        } else {
            print("The binary search tree cannot have the same value.")
        }
    }
}
~~~

### isConatin(_:)
#### Logic
* Root Node 존재 유무 확인
  * 존재하지 않는다.
    * BST를 구성하는 노드들은 루트노드를 제외하고 모두 부모 노드를 갖는다. 따라서 Root Node가 없다면 BST는 비어있다는 의미이다. 
  * 존재한다.
    * 다음 분기로 이동
* 현재 노드의 위치를 확인할 수 있는 변수 할당
* 반복문
  * value와 현재 노드 값이 같음? -> return true
  * value가 현재 노드 값보다 작음
    * 왼쪽 자식 노드 존재 유무
      * 존재하지 않음 -> 현재 노드가 리프 노드인데 value와 같은 값을 찾지 못함 -> 존재하지 않는 Node이므로 return false
      * 존재함 -> 현재 노드가 현재 노드의 왼쪽 자식 노드의 참조를 가리키도록 설정
  * value가 현재 노드 값보다 큼
    * 오른쪽 자식 노드 존재 유무
      * 존재하지 않음 -> 현재 노드가 리프 노드인데 value와 같은 값을 찾지 못함 -> 존재하지 않는 Node이므로 return false
      * 존재함 -> 현재 노드가 현재 노드의 오른쪽 자식 노드의 참조를 가리키도록 설정
  * value와 현재 노드의 값이 같음
    * return true

#### Code
~~~swift
// file: "BinarySearchTree.search(_ value: T).swift"
func search(_ value: T) -> Bool {
    guard let node = self.root else { return false }
    var crtNode = node
        
    while true {
        if value == crtNode.value {
            return true
        } else if value < crtNode.value {
            guard let leftNode = crtNode.left else { return false }
            crtNode = leftNode
        } else if value > crtNode.value {
            guard let rightNode = crtNode.right else { return false }
            crtNode = rightNode
        }
    }
        
    return false
}
~~~


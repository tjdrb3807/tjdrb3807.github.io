---
layout: post
title: "[Data Structure] Hash Table"
sitemap: false
# image: /assets/img/blog/thumbnail/etc/git_section07.png
categories: [algorithm, data-structure]
tags: [data-structure]
# description: App이 Foreground 또는 Background 상태에 있을 때 시스템 Notification에 응답하고 기타 중요 시스템관련 이벤트를 처리한다.
---

* toc
{:toc}

## Introduce
해시 테이블은 Key를 Value에 매핑할 수 있는 구조로, 연관 배열 추가에 사용되는 자료 구조이다.    
해시 테이블은 Key를 기반으로 해시 함수를 통해 해쉬값(해시 테이블 bucket의 고유 index)으로 변환하고 특정 bucket에 위치한 데이터에 접근할 수 있다.   
해시 테이블은 Key를 통해 Value를 한 번에 찾을 수 있으므로 삭제, 삽입, 검색이 평균적으로 O(1)이 걸리는 효율적인 자료구조이다.    
하지만 해시 함수를 통해 Key값을 해시값으로 변환하는 과정에서 충돌을 막기 위해 일반적으로 큰 공간복잡도가 필요하며, 충돌이 발생한 최악의 경우 O(n)의 시간복잡도가 발생할 수 있다.

## Hash Function
* 해시 테이블을 자료구조로 구현하기 위해선, Key 값을 정수값으로 매핑하는 과정이 필요하다. 
* Key 값을 테이블의 index로 매핑하는 것이 해시 함수이며, 해시 함수의 결과값을 해시, 해시 값, 해시 코드라 칭한다.
* 해시 함수는 특정 해싱 알고리즘을 적용해, 임의의 크기를 갖는 데이터를 고정된 크기의 값으로 매핑하는 함수다.
* Key 값을 통해 해시값을 얻을 수 있지만, 해시 값에서 Key 값을 얻을 수 없으므로 해시함수는 단방향 구조를 이룬다.

~~~swift
func hash(for key: String) -> Int {
    let unicodeScalrs = key.unicodeScalars.map { Int($0.value) }

    return unicodeScalrs.reduce(0, +) & hashTable.size
}
~~~

* 위 코드처럼 해시 함수는 최대한 유니크하게 해시 테이블에 매핑해주어야 한다.
* 해시 테이블의 크기는 한정되어있지만, 함수의 정의역 공간이 너무 크기 때문에, 다른 Key 값임에도 해시 값이 겹치는 결과가 발생하게 된다.
* 이를 해시 충돌이라 칭한다.
* 해시 충돌은 불가피하지만, 발생할수록 해시테이블 자료구조의 성능이 저하되기 때문에 최대한 피해야한다.
* 즉, 해시 함수는 충돌을 최소화하기 위해 해시 테이블에 균등한 분포로 매핑해주어야 하며, 계산하기 쉬워야한다.

~~~swift
return unicodeScalrs.reduce(0, +) & hashTable.size
~~~

* 또한, 해시 함수에서 최종적으로 해시 테이블의 사이즈(bucket의 수 )로 모듈러 연산을 진행해 주는데 이떄, 나눠주는 값이 짝수가 된다면, 내부적으로 저장되는 이진수의 비트다 Shift되어 기존의 해시값이 사라진 수 있다.
* 따라서 해시 테이블의 사이즈는 되도록이면 20이상의 소수를 사용한다.
* 해시 함수 내부에서 곱셈 혹은 나눗셈 연산을 진행할 때 역시 마찬가지로 20 이상의 소수를 사용한다.

## 해시 테이블의 충돌
해시 함수를 통해 Key를 해싱했을 때 해시 값이 동일하게 나오는 경우의 수 때문에 충돌이 발생한다.    
(Key는 다르지만, 해시 함수가 단순해서 결과값인 해시값이 겹치는 문제)    
위와 같은 경우, Key-Value가 덮어씌어지는 충돌 현상이 발생한다.    
충돌을 해결하기 위해 가장 대표적인 두 가지 알고리즘이 존재한다.

### Chaining(=닫힌 주소)
충돌이 일어날 경우, Linked List를 활용해 데이터를 추가로 뒤에 연결 시켜서 저장하는 기법

### Linear Probing 기법(=오픈 주소)
충돌이 일어날 경우, 해당 해시값부터 순회하며 가장 처음 나오는 빈 공간에 저장하는 기법
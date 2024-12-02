---
layout: post
title: "2022-12-02 TIL"
sitemap: false
categories: [study, til]
tags: [til]
---

* toc
{:toc}

## 알고리즘 코드카타 - 프로그래머스: 올바른 괄호
### 문제 설명
괄호가 바르게 짝지어졌다는 것은 '(' 문자로 열렸으면 반드시 짝지어서 ')' 문자로 닫혀야 한다는 뜻입니다. 예를 들어

* "()()" 또는 "(())()" 는 올바른 괄호입니다.
* ")()(" 또는 "(()(" 는 올바르지 않은 괄호입니다.

'(' 또는 ')' 로만 이루어진 문자열 s가 주어졌을 때, 문자열 s가 올바른 괄호이면 true를 return 하고, 올바르지 않은 괄호이면 false를 return 하는 solution 함수를 완성해 주세요.

### 제한 사항
* 문자열 s의 길이 : 100,000 이하의 자연수
* 문자열 s는 '(' 또는 ')' 로만 이루어져 있습니다.

### 풀이
~~~swift
import Foundation

func solution(_ s: String) -> Bool {
    var ans: Bool = false
    var stack = Stack<Character>()
    
    for i in s {
        if i == "(" {
            stack.items.append(i)
        } else {
            guard let top = stack.items.last, top == "(" else { return false }
            stack.pop()
        }
    }
    
    ans = stack.items.isEmpty
    return ans
}

struct Stack<T> {
    var items: [T] = []
    
    mutating func push(_ item: T) { items.append(item) }
    
    mutating func pop() { 
        guard !items.isEmpty else { return }
        items.popLast() 
    }
}
~~~
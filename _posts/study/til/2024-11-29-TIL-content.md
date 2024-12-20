---
layout: post
title: "2022-11-29 TIL"
sitemap: false
categories: [study, til]
tags: [til]
---

* toc
{:toc}

## 알고리즘 코드카타 - 프로그래머스: 타겟 넘버
### 문제 설명
n개의 음이 아닌 정수들이 있습니다. 이 정수들을 순서를 바꾸지 않고 적절히 더하거나 빼서 타겟 넘버를 만들려고 합니다. 예를 들어 [1, 1, 1, 1, 1]로 숫자 3을 만들려면 다음 다섯 방법을 쓸 수 있습니다.

~~~swift
-1+1+1+1+1 = 3
+1-1+1+1+1 = 3
+1+1-1+1+1 = 3
+1+1+1-1+1 = 3
+1+1+1+1-1 = 3
~~~

사용할 수 있는 숫자가 담긴 배열 numbers, 타겟 넘버 target이 매개변수로 주어질 때 숫자를 적절히 더하고 빼서 타겟 넘버를 만드는 방법의 수를 return 하도록 solution 함수를 작성해주세요.

### 제한 사항
* 주어지는 숫자의 개수는 2개 이상 20개 이하입니다.
* 각 숫자는 1 이상 50 이하인 자연수입니다.
* 타겟 넘버는 1 이상 1000 이하인 자연수입니다.

### 풀이
~~~swift
import Foundation

func solution(_ numbers:[Int], _ target:Int) -> Int {
    var queue = [(Int, Int)]()
    queue.append((-numbers[0], 0))
    queue.append((numbers[0], 0))
    var answer = 0

    while !queue.isEmpty {
        let q = queue.removeLast()
        let num = q.0
        var idx = q.1
        idx += 1
        if idx < numbers.count {
            queue.append((num+numbers[idx], idx))
            queue.append((num-numbers[idx], idx))
        } else {
            if target == num {
                answer += 1
            }
        }
    }

    return answer
}

let numbers = [1, 1, 1, 1, 1]
let target = 3

print(solution(numbers, target))
~~~



---
title:  "[스위프트][LeetCode][Medium] Fruit Into Baskets (투 포인터) " 

categories:
  - Leet Code
tags:
  - [Algorithm, Coding Test, 스위프트, 투 포인터]

toc: true
toc_sticky: true

date: 2022-05-28
last_modified_at: 2022-05-28
---

## 🧞‍♂️ 난이도 

> Medium

<br>

## 🧞‍♂️ 문제

> <https://leetcode.com/problems/fruit-into-baskets/>

<br>

## 🧞‍♂️ 풀이 방법

투 포인터(슬라이딩 윈도우)를 활용해서 풀면 된다.

그리고 2가 됐을때
1. `right`는 계속 한칸씩 오른쪽으로 이동한다.
    - 총 과일의 수를 기록한다
    - 고유한 숫자(과일)의 `count`를 계속해서 갱신한다.
2. 고유한 숫자의 개수가 2가 됐을때 다시 1이 될때까지 `left`를 왼쪽으로 이동시킨다.
    - 이때 고유한 과일들도 바꿔줘야한다.

## 🧞‍♂️ 스위프트 코드
```swift
---
title:  "[스위프트][LeetCode][Medium] Fruit Into Baskets (투 포인터) " 

categories:
  - Leet Code
tags:
  - [Algorithm, Coding Test, 스위프트, 투 포인터]

toc: true
toc_sticky: true

date: 2022-05-28
last_modified_at: 2022-05-28
---

## 🧞‍♂️ 난이도 

> Medium

<br>

## 🧞‍♂️ 문제

> <https://leetcode.com/problems/fruit-into-baskets/>

<br>

## 🧞‍♂️ 풀이 방법

투 포인터(슬라이딩 윈도우)를 활용해서 풀면 된다.

그리고 2가 됐을때
1. `right`는 계속 한칸씩 오른쪽으로 이동한다.
    - 총 과일의 수를 기록한다
    - 고유한 숫자(과일)의 `count`를 계속해서 갱신한다.
2. 고유한 숫자의 개수가 2가 됐을때 다시 1이 될때까지 `left`를 왼쪽으로 이동시킨다.
    - 이때 고유한 과일들도 바꿔줘야한다.

## 🧞‍♂️ 스위프트 코드
```swift
fileprivate extension Dictionary where Key == Int, Value == Int {
    func contains(_ number: Int) -> Bool {
        let value = self[number, default: 0]
        
        return value == 0 ? false : true
    }
}

class Solution {
    func totalFruit(_ fruits: [Int]) -> Int {
        var maxAmount = 0
        var left = 0
        var dictionary: [Int: Int] = [:]
        var dictionaryCount = 0
        var fruitCount = 0
        
        for right in 0..<fruits.count {
            if dictionaryCount == 2 && !dictionary.contains(fruits[right]) {
                while left < right {
                    dictionary[fruits[left]]! -= 1
                    fruitCount -= 1
                    left += 1
                    if dictionary[fruits[left-1]]! == 0 {
                        dictionaryCount -= 1
                        break
                    }
                }
            }
            let value = dictionary[fruits[right], default: 0]
            
            if value == 0 {
                dictionaryCount += 1
            }
            dictionary[fruits[right], default: 0] = value + 1
            fruitCount += 1
            maxAmount = max(maxAmount, fruitCount)
        }
        return maxAmount
    }
}
```
***
<br>

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}

```
***
<br>

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}

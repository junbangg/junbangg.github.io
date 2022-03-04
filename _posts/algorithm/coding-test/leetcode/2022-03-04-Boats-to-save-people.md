---
title:  "[스위프트][Leet Code - Boats to Save People][Medium] (연결리스트)" 

categories:
  - leetcode
tags:
  - [Algorithm, Coding Test, Swift, LeetCode, 그리디, 투포인터]

toc: true
toc_sticky: true

date: 2022-03-04
last_modified_at: 2022-03-04
---

## 🧞‍♂️ 난이도 

> Medium

<br>

## 🧞‍♂️ 문제
**Boats to Save People**
> <https://leetcode.com/problems/boats-to-save-people/>

## 🧞‍♂️ 유형
> 투포인터, 그리디

<br>

## 🧞‍♂️ 내 풀이

이 문제는 처음에 그리디하게 풀려다가 안돼서
투포인터를 썼는데 또 잘 안됐는데
두 개념을 합해서 풀어야 풀렸다

배열을 정렬 시켜서 양쪽 끝의 값을 하나의 배에 넣는다는 로직이다.

두 무게를 합했을때 `limit` 보다 작거나 같으면
왼쪽 포인터를 증가시키고, 그게 아니면 무조건 오른쪽 포인터를 감소시킨다.


***

## 🧞‍♂️ 스위프트 포인터 풀이
```swift
class Solution {
    func numRescueBoats(_ people: [Int], _ limit: Int) -> Int {
        let people = people.sorted(by: <)
        var answer = 0
        var left = 0, right = people.count - 1
        
        while left <= right {
            if people[left] + people[right] <= limit {
                left += 1
            }
            right -= 1
            answer += 1    
        }
        return answer
    }
}

```



<br>

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}



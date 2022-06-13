---
title:  "[스위프트][Leet Code][Medium] Wiggle Subsequence (그리디)" 

categories:
  - Programmers
tags:
  - [Algorithm, Coding Test, 스위프트, 그리디]

toc: true
toc_sticky: true

date: 2022-06-13
last_modified_at: 2022-06-13
---

## 🧞‍♂️ 난이도 

> Medium

<br>

## 🧞‍♂️ 문제

> <https://leetcode.com/problems/wiggle-subsequence/>

<br>

## 🧞‍♂️ 풀이 방법

주어진 배열에서 

연속된 두 수의 차이가 
<br>
+, -, +, -, +...

또는
<br>
-, +, -, +, -, +...
<br>
이런식으로 왔다리갔다리 하는걸 `Wiggle Subsequence`라고 하는데, 
중간중간에 숫자를 빼서 만들 수 있는 최대 길이의 `Wiggle Subsequence` 를 만들어야합니다.


이 문제는 주어진 예제 케이스를 그려보면 대충 `O(N)` 로 풀 수 있는 아이디어가 떠오릅니다.

다음 배열을 
<br>
`[1,17,5,10,13,15,10,5,16,8]` 
`WiggleSubsequence` 로 표시해보면 다음과 같습니다.

```swift
     -          -  -    -
[1,17,5,10,13,15,10,5,16,8]
  +    +  +  +       +
```

문제 풀이는 간단합니다. 연속으로 `+`, `-` 가 있어도, 아무거나 빼버리면 `Wiggle Subsequence`를 만들 수 있기 때문에 그냥 **그리디하게 연속된 `+`, 또는 `-` 묶음의 개수를 세면 됩니다.**

즉, 여기서는 (1-17), (17-5), (5-10, 10-13, 13-15) (15-10, 10-15), (5-16), (16-18) 총 6 덩어리가 있고, 문제에서는 시작 숫자부터 끝 숫자의 길이이기 때문에 `+1`을 해주면 됩니다.



## 🧞‍♂️ 스위프트 코드
```swift
class Solution {
    func wiggleMaxLength(_ nums: [Int]) -> Int {
        //edge case
        if nums.count == 1 {
            return nums.count
        }
        
        var currentState: Bool? = nil
        var wiggleCount = 0
        
        for i in 1..<nums.count {
            if nums[i] == nums[i-1] {
                continue
            }
            let isPositive = nums[i] - nums[i-1] > 0
            
            if isPositive == currentState {
                continue
            } else {
                currentState = isPositive
                wiggleCount += 1
            }
        }
        
        return wiggleCount + 1
    }
}
```

## 테스트 케이스
```swift
[2, 1] -> 2
[1, 2] -> 2
[1, 1] -> 1
[1] -> 1
[3, 3, 3, 0, 3] -> 3
```
***
<br>

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}

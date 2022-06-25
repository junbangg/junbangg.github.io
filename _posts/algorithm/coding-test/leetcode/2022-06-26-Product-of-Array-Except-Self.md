---
title:  "[스위프트][Leet Code - Product of Array Except Self][Easy] (누적합)" 

categories:
  - leetcode
tags:
  - [Algorithm, Coding Test, Swift, LeetCode, 누적합]

toc: true
toc_sticky: true

date: 2022-06-25
last_modified_at: 2022-06-25
---

## 🧞‍♂️ 난이도 

> Medium

<br>

## 🧞‍♂️ 문제
**Product of Array Except Self**
> <https://leetcode.com/problems/product-of-array-except-self/>

## 🧞‍♂️ 유형
> 누적합

<br>

## 🧞‍♂️ 내 풀이

처음 풀었을때는 그냥 살짝 어거지로 풀었는데 찾아보니까 옛날에 깔끔한 누적합으로 풀었던 기록이 있었다

## 어거지 풀이
어거지로 풀때는 세 가지 경우로 나눠서 생각해서 문제를 풀었다.
1. 0이 없을때
    - `total` = 모든 숫자를 곱한 값
    - 모든 숫자를 `total` 에서 나눈 값을 저장한다.
3. 0이 하나 일때
    - `total`을 구할때 0은 곱하지 않는다.
    - 0이 아닌 숫자는 전부 0으로 저장하고, 0인 숫자는 `total`로 저장한다.
5. 0이 두개 이상일때
    - 그냥 `[0] x numbers.count` 가 정답이다.

## 어거지 풀이 코드 (스위프트)
```swift
class Solution {
    func productExceptSelf(_ numbers: [Int]) -> [Int] {
        var answer: [Int] = Array(repeating: 0, count: numbers.count)
        var total = 1
        var zeroCount = 0

        for number in numbers {
            if number == 0 {
                zeroCount += 1
            } else {
                total *= number
            }
        }

        for (index, number) in numbers.enumerated() {
            var inputNumber = 0

            if zeroCount == 0 {
                inputNumber = total / number
            } else if zeroCount == 1 && number == 0 {
                inputNumber = total
            }
            answer[index] = inputNumber
        }

        return answer
    }
}
```

## 누적곱 풀이
테스트 케이스: [1, 2, 3, 4]
결과: [24, 12, 8, 6]
1. 오른쪽으로 한칸 밀린? 입력 배열에 대한 누적곱? 배열을 만든다
    - [1, 2, 3]에 대한 누적 곱 배열을 만든다. 즉, 마지막 값인 4는 곱해지지않는다.
    - [1, 1, 2, 6]
    - 이때, 누적 곱의 내용을 까보면 다음과 같다.
    - [1, 1, 1x2, 1x2x3]
2. 1부터 시작해서 입력 배열의 값을 뒤에서부터 하나씩 곱해서 누적곱 배열에 값들에 곱한다.
```
입력 배열: [1, 2, 3, 4]

[1,       1,     2,    6] (스텝1에서 만든 누적곱 배열)
                       1  (1부터 시작)
                1x4       (입력 배열의 마지막 값)
        1x4x3             (거기에 뒤에서 두번째 값 곱함)
1x4x3x2                   (...)
```
이렇게 되면 `[2x3x4, 1x3x4, 1x2x4, 1x2x3]` 로 자신을 제외한 곱들이 쌓인 정답이 나온다. 

## 누적곱 풀이 코드
```swift
class Solution {
    func productExceptSelf(_ numbers: [Int]) -> [Int] {
        var prefix: [Int] = []
        var previousNumber = 1
        
        for number in numbers {
            prefix.append(previousNumber)
            previousNumber *= number
        }
        
        previousNumber = 1
        
        for i in stride(from: numbers.count-1, to: -1, by: -1) {
            prefix[i] *= previousNumber
            previousNumber *= numbers[i]
        }
        
        return prefix
    }
}
```



<br>

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}



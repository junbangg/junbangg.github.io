---
title:  "[스위프트][Leet Code - Simplify Path][Medium] (Stack)" 

categories:
  - leetcode
tags:
  - [Algorithm, Coding Test, Swift, LeetCode, Stack]

toc: true
toc_sticky: true

date: 2022-5-14
last_modified_at: 2022-5-14
---

## 🧞‍♂️ 난이도 

> Medium

<br>

## 🧞‍♂️ 문제
**Simplify Path**
> <https://leetcode.com/problems/simplify-path/>

## 🧞‍♂️ 유형
> 자료구조(Stack)

<br>

## 🧞‍♂️ 풀이

이 문제는 문제를 이해하기만 하면 금방 풀 수 있습니다.
주요 로직은 다음과 같습니다.

1. `/` 를 기준으로 `split` 한다.
    - `Swift`의 `split(separator: "/")` 를 사용하면 `//` 도 처리해준다
2. `.` 처리
    - `.` 가 나오면 현재 이건 `absolute path`에서 현재 `directory`를 의미하는 것이기 때문에 `canonical path` 에는 포함 안시키면 된다.
3. '..' 처리
    - 윗 directory로 가는 커맨드이기 때문에 `stack`의 마지막 요소를 `pop` 한다.
4. 그외의 제외한 모든 데이터는 스택에 때려넣는다.
5. 마지막에 `stack`에 남아있는 걸 `join` 해서 반환한다.
6. 
***

## 🧞‍♂️ 사용했던 테스트 케이스

```swift
// 제공된 TC
let tc1 = "/home/"
let tc2 = "/../"
let tc3 = "/home//foo/"
// 생각해본 TC
let tc2 = "/home//.../olaf/.././_"
    -> "/home/.../_"
let tc3 = "/home/../test/../current/asdf/.././"
    > "/current"
```

## 🧞‍♂️ 스위프트 풀이
```swift
class Solution {
    func simplifyPath(_ path: String) -> String {
        var stack: [String] = []
        
        for subPath in path.split(separator: "/") {
            if subPath == "." {
                continue
            } else if subPath == ".." {
                if !stack.isEmpty {
                    stack.removeLast()
                }
            } else {
                stack.append(String(subPath))
            }
        }
        return "/\(stack.joined(separator: "/"))"
    }
}
```




<br>

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}



---
title:  "[파이썬, 스위프트][Leet Code - Reverse Linked List][Easy] (연결리스트)" 

categories:
  - leetcode
tags:
  - [Algorithm, Coding Test, Python, Swift, LeetCode, 연결리스트]

toc: true
toc_sticky: true

date: 2021-12-28
last_modified_at: 2021-12-28
---

## 🧞‍♂️ 난이도 

> Easy

<br>

## 🧞‍♂️ 문제
**최소 회의실 개수**
> <https://leetcode.com/problems/reverse-linked-list/>

## 🧞‍♂️ 유형
> 연결리스트

<br>

## 🧞‍♂️ 내 풀이

이 문제는 크게 **재귀**, **포인터를 활용한 반복문** 두가지 방법이 있다.
기본적인 문제지만 오랜만에 볼때마다 헷갈려서 
정리 해둬야겠다.

### 포인터 풀이
`reversedHead` 를 하나 지정해놓고
연결리스트 앞에서 뒤까지 반복문으로 이동하면서
포인터의 방향을 반대로 뒤집어주면 된다.

이때의 주요 로직은
```python
# 다음으로 이동할 노드를 temp에 저장
temp = current.next
# 현재 노드가 가리키는 포인터를 reversedHead를 가리키도록 변경(반대로 뒤집는다)
current.next = reversedHead
# reversedHead를 현재 노드로 지정해준다
reversedHead = current
# 위에서 잠깐 저장해뒀던 다음 노드로 현재 노드를 이동시킨다
current = temp
```


### 재귀 풀이

재귀 풀이도 마찬가지로 `head`, `previous` 를 이용해서 재귀적으로 포인터를 뒤집는 방법이다.

주요 로직은 다음고 같다.
1. **base case** 는 `if head == nil ` 로 해놓고 걸리게면, `previous` 를 리턴하면 그게 뒤집혀진 연결리스트가 될것이다.
2. 인자로 들어온 `head` 의 `.next` 를 직전 노드로 들어온 `previous` 로 설정하면서 연결리스트의 방향을 뒤집는다.
3. `recurse(next, head)` 를 호출하면서, 앞으로 이동한다.


***

## 🧞‍♂️ 파이썬 포인터 풀이
```python
class Solution:
    def reverseList(self, head: ListNode) -> ListNode:
        cur, rev = head, None
        while cur:
            temp, cur.next = cur.next, rev
            rev, cur = cur, temp
        return rev
```
## 🧞‍♂️ 스위프트 포인터 풀이
```swift
class Solution {
    func reverseList(_ head: ListNode?) -> ListNode? {
        var current = head
        var reversedHead: ListNode? = nil
        
        while current {
            let temp = current?.next
            current!.next = reversedHead
            reversedHead = current
            current = temp
        }
        
        return reversedHead
    }
}

```
## 🧞‍♂️ 파이썬 재귀 풀이
```python
class Solution:
    def reverseList(self, head: ListNode) -> ListNode:
        def recurse(head, previous=None):
            if not head:
                return previous
            next, node.next = node.next, previous
            return recurse(next, node)
        
        return recurse(head)
```

## 🧞‍♂️ 스위프트 재귀 풀이
```swift
class Solution {
    func recurse(_ head: ListNode?, _ previous: ListNode? = nil) -> ListNode? {
        var head = head
        if head == nil {
            return previous
        }
        let next = head?.next
        head!.next = previous

        return recurse(next, head)
    }

    func reverseList(_ head: ListNode?) -> ListNode? {
        return recurse(head)
    }
}
```




<br>

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}



---
title:  "Swift 코딩테스트 Cheatsheet - 문자열 편" 

categories:
  - swift
tags:
  - [iOS, swift, 코딩테스트, cheatsheet]

toc: true
toc_sticky: true

date: 2021-12-12
last_modified_at: 2020-12-12
---

# Integer -> Binary String

```swift

let N = String(N, radix: 2)
// Binary String ->  Integer
let N = Int(strtoul("1111000", nil, 2))
```
# Array indexing
```swift
//Insert
A.insert("somthing", at: 0)
```

# Access Last element
```swift
A.last
```

# Count element Frequencies in Array
**파이썬** `Counter` 와 똑같은 기능
```swift
let items = ["a","b","c","c"]
let mappedItems = items.map{ ($0, 1) }
let counts = Dictionary(mappedItems, uniquingKeyswith: +)
```

# Count unique values in Array
```swift
return Set(array).count
```

# `String` 에서 `index` 하나 빼고 `substring` 리턴하는 방법
```swift
func getSubstring(of string: String, without deleteIndex: Int) -> String {
    var substring = string
    let index = substring.index(substring.startIndex, offsetBy: deleteIndex)
    substring.remove(at: index)
    return substring
}
```

# Substring Extension

문자열 파이썬 처럼 다루기 편하게 해주는 문자열 `extension` 
**근데 이거 코테때 잘 안됐었다**
Reference: https://stackoverflow.com/a/46634511
```swift
public extension String {
  subscript(value: Int) -> Character {
    self[index(at: value)]
  }
}

public extension String {
  subscript(value: NSRange) -> Substring {
    self[value.lowerBound..<value.upperBound]
  }
}

public extension String {
  subscript(value: CountableClosedRange<Int>) -> Substring {
    self[index(at: value.lowerBound)...index(at: value.upperBound)]
  }

  subscript(value: CountableRange<Int>) -> Substring {
    self[index(at: value.lowerBound)..<index(at: value.upperBound)]
  }

  subscript(value: PartialRangeUpTo<Int>) -> Substring {
    self[..<index(at: value.upperBound)]
  }

  subscript(value: PartialRangeThrough<Int>) -> Substring {
    self[...index(at: value.upperBound)]
  }

  subscript(value: PartialRangeFrom<Int>) -> Substring {
    self[index(at: value.lowerBound)...]
  }
}

private extension String {
  func index(at offset: Int) -> String.Index {
    index(startIndex, offsetBy: offset)
  }
}
```

## 사용 방법
```swift
let text = "Hello world"
text[0] // H
text[...3] // "Hell"
text[6..<text.count] // world
text[NSRange(location: 6, length: 3)] // wor
```

# String -> Array
```swift
let str = "aBcDeF"

// Character형을 요소로 갖는 배열
let charArr = str.map {$0}
print("\(charArr) : \(type(of: charArr))") // ["a", "B", "c", "D", "e", "F"] : Array<Character>


// String형을 요소로 갖는 배열
let strArr = str.map {String($0)}
print("\(strArr) : \(type(of: strArr))") // ["a", "B", "c", "D", "e", "F"] : Array<String>
```

# Array -> String

배열을 문자열로 변환
```swift
let array = ["A", "B", "C"]
let str1 = String(Array)
let str2 = array.joined(separator: "")
let str3 = array.reduce("", +)
```

# 접두어 접미어 확인

```swift
let str = "aBcDeF"

// 접두어 (앞에서부터 몇 글자)
print(str.prefix(3)) // aBc
print(str.prefix(4)) // aBcD

// 접미어 (뒤에서부터 몇 글자)
print(str.suffix(1)) // F
print(str.suffix(2)) // eF


let str = "aBcDeF"

// 접두어 (앞에서부터 몇 글자)
print(str.hasPrefix("a"))  // true
print(str.hasPrefix("aB")) // true
print(str.hasPrefix("ab")) // false

// 접미어 (뒤에서부터 몇 글자)
print(str.hasSuffix("F"))  // true
print(str.hasSuffix("DeF")) // true
print(str.hasSuffix("FeD")) // false

```

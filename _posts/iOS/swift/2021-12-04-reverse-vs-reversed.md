---
title:  "reverse() vs reversed()" 

categories:
  - swift
tags:
  - [iOS, swift]

toc: true
toc_sticky: true

date: 2021-12-04
last_modified_at: 2020-12-04
---

# reverse() vs reversed()

스위프트 배열을 역순으로 뒤집고 싶을때 `swift` 에서 제공하는 방법은 2가지가 있다.

### `reverse()`

`reverse` 같은 경우는 원소들이 `in place` 로 뒤집혀진다. 즉, 해당 배열 내부적으로 값들을 바꾸는 것이다.

`@mutating` 으로 정의되어 있다고 생각하면 된다.

그리고 또 다른 대표적인 특징은 **시간 복잡도가 `O(N)` 이라는 것이다.**

### `reversed()`

```swift
// 공식문서에 나와있는 String 을 뒤집는 방법
let reversedWord = String(word.reversed())
print(reversedWord)
// Prints "sdrawkcaB"
```

반대로 `reversed` 같은 경우는 뒤집혀진 배열이 하나 생성된다.

즉, `in place` 가 아니라, 새로운 배열을 만들어준다는 뜻이다.

그런데 여기서 특이한 점은, `reverse` 와 반대로 `reversed` 는 **시간 복잡도가 `O(1)` 이라는 것이다.**

🤔 곰곰히 생각해보니까 이게 말이 되나 싶다..

아무리 생각해도 배열을 뒤집는데 상수시간으로 가능 하지가 않을것 같은데 말이다.

그래서 공식문서를 조금 더 자세하게 보니까

![image](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9d4329ca-f04f-4301-bdaf-1f971edf37e1/Screen_Shot_2021-12-04_at_1.02.39_AM.png)

`ReversedCollection` 이라는 `Generic Struct` 이 생성된다.

![image](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b9fa033d-c5ee-4237-818a-6568eadbad6b/Screen_Shot_2021-12-04_at_1.04.55_AM.png)

내가 이해한게 맞다면, 결론적으로 봤을때 `**reversed` 는 새로운 배열을 만들지도 않고, 사실 뒤집지도 않는다.**

즉, `reversed` 를 사용하면, 해당 배열을 `ReversedCollection` 이라는 구조체로 `wrapping` 을 한다.

그리고 `ReversedCollection` 은 안에 있는 배열을 "반대로도" 들여다볼 수 있는 기능성을 제공해주는것 같다.

그렇기 때문에 `reversed` 의 시간 복잡도도 `O(1)` 인것 같다.

시간적 효율, 공간적 효율을 모두 잡기 위해 있는 방법인것 같다.

## Reference

<[https://developer.apple.com/documentation/swift/array/1690025-reversed](https://developer.apple.com/documentation/swift/array/1690025-reversed)>

<[https://stackoverflow.com/questions/46626189/swift-array-reversed-function-o1-complexity](https://stackoverflow.com/questions/46626189/swift-array-reversed-function-o1-complexity)>
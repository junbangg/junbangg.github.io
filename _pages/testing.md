---
title: Portfolio
layout: collection
permalink: /portfolio/
collection: portfolio
entries_layout: grid
classes: wide
---

# ARC

**ARC**(Automatic Reference Counting) 는 자동으로 메모리를 관리해주는 방식을 말한다.

- 레퍼런스의 Instance가 메모리 공간에 할당되면 레퍼런스 카운트가 1 증가
- Instance 가 해제되면 레퍼런스 카운트가 1 감소
- 레퍼런스 카운트가 0이 되면 자동으로 레퍼런스 메모리를 해제시킨다.

즉 참조 카운팅이 0이 될때에만 메모리에서 해제된다는 뜻.

- compile time 때 해준다
- 모든걸 자동으로 해주는건 아니고
    - retain/release 코드를 사이사이에 넣어주는 것.
    - 이게 어느 위치/시점에 들어가는지를 이해하기 위해 ARC를 이해하기 위함
- "automatic" 이 혼란을 줄 여지가 있음(?)

iOS가 Garbage Collection을 사용하는 안드로이드에 비해서 조금 더 효율적이라는 평가를 받을 수 있는것도 ARC 덕분이다.

[What exactly makes iOS more efficient?](https://www.quora.com/What-exactly-makes-iOS-more-efficient/answer/Franklin-Veaux)

**What exactly makes iOS more efficient?**

Franklin Veaux's answer: One of the biggest efficiencies in iOS compared to Android is memory management. iOS uses a memory management technique called ‘automatic reference counting.’ Android uses a technique called ‘garbage collection.’ Garbage collection requires no special skill on the devel...

www.quora.com

ARC 가 가져다주는 장점은, **메모리 해제의 예측이 가능해서 관리를 해줄 수 있다는 점**이다.

반면에 Garbage Collection 은 성능 저하의 가능성이 불가피한 상황이 발생 할 수 있고, 인스턴스들이 언제 메모리에서 해제 될지 예측하기 어렵다.

- GC는 다 갖다 버린다(최대한 보수적으로 가져갈 수 밖에 없음) 애매하다 싶은것들은 남긴다. 프로그래머들이 제이를 할 수 없다.

**동작원리:** 클래스의 새로운 인스턴스를 만들 때 ARC 가 메모리 할당을 해준다. 그리고 인스턴스가 더 이상 사용 되지 않는다고 판단 할 때(참조 카운팅이 0일 때) 메모리를 해제한다. 레퍼런스 프로퍼티에 인스턴스를 할당하면 ARC 는 참조되는 프로퍼티의 갯수를 카운팅하여 참조하는 모든 변수가 인스턴스를 해제하기 전에는 ARC 는 인스턴스를 메모리에서 해제하지 않는다.

**이렇게 동작하는 이유는, 아래 글에서도 나와있듯이, 어떠한 인스턴스, 또는 그 인스턴스의 속성들에 접근을 해야되는데, 그 인스턴스가 메모리에서 해제되었으면 오류가 발생할것이다. 그래서 ARC 가 그 인스턴스를 가리키는 다른 인스턴스의 개수를 세다가, count의 수가 없을때(0개 일때) 메모리에서 해제하는 것이다.**

**To make sure that instances don’t disappear while they are still needed**, ARC tracks how many properties, constants, and variables are currently referring to each class instance. ARC will not deallocate an instance as long as at least one active reference to that instance still exists.

To make this possible, whenever you assign a class instance to a property, constant, or variable, that property, constant, or variable makes a *strong reference* to the instance. The reference is called a “strong” reference because it keeps a firm hold on that instance, and does not allow it to be deallocated for as long as that strong reference remains.

[Automatic Reference Counting - The Swift Programming Language (Swift 5.5)](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html)

- 참조 카운팅은 클래스의 인스턴스만 해당함. (구조체와 Enumeration 은 value type 이지 reference type 이 아니라서 레퍼런스로 저장 되는게 아님)

```swift
class Person {
    let name : String
    init(name: String) {
        self.name = name
        print("\(name)is being initialized")
    }
    deinit {
        print("\(name)is being deinitialized")
    }
}
/*
  변수들이 Optional(Person?, not Person) 이기 때문에 자동으로 nil 값으로 생성된다, 그래서 처음부터 Person 인스턴스를 참조 하지 않는다.
 */
var reference1 : Person?
var reference2 : Person?
var reference3 : Person?

reference1 = Person(name: "June")
reference2 = reference1
reference3 = reference1
//이러면 3개의 "Strong" 참조가 발생한다
reference3 = nil
reference2 = nil
//이렇게 해도 하나의 Strong 참조가 존재하기 때문에 ARC 는 메모리에서 해제를 하지 않는것이다
reference1 = nil
//이렇게 까지 해야 없어진다
```

**Strong Reference Cycle:**

**두개 의 클래스 인스턴스가 서로에 대한 strong 참조를 하고 있어서 ARC 가 메모리 해제를 못하는 상황. (reference count 가 0개가 될 수 없기 때문에)**

예시

```swift
class Person {
    let name: String
    init(name: String) { self.name = name }
    var apartment: Apartment?
    deinit { print("\(name) is being deinitialized") }
}

class Apartment {
    let unit: String
    init(unit: String) { self.unit = unit }
    var tenant: Person?
    deinit { print("Apartment \(unit) is being deinitialized") }
}

var john : Person?
var unit4A : Apartment?

john = Person(name: "John Appleseed")
unit4A = Apartment(unit: "4A")

//여기서 strong reference cycle 이 발생
john!.apartment = unit4A
unit4A!.tenant = john
```

즉! 여기서

![Screen Shot 2021-11-21 at 10.27.53 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/91fb6b41-82d7-4c47-be20-1c913f04288c/Screen_Shot_2021-11-21_at_10.27.53_PM.png)

```swift
john = nil
unit4A = nil
```

을 해도!

![Screen Shot 2021-11-21 at 10.28.28 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/daca95c1-1018-44ab-8514-b67aef0ce3d6/Screen_Shot_2021-11-21_at_10.28.28_PM.png)

이렇게 **순환참조** 가 발생

**해결 방안**: 클래스 간의 관계를 `strong` 이 아닌 `weak`, `unowned` 로 지정. 그럼 순환참조 없이 인스턴스들이 서로 참조 할 수 있다.
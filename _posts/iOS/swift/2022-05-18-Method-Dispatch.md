---
title:  "Swift의 Method Dispatch 파헤쳐보기 part 1" 

categories:
  - swift
tags:
  - [iOS, swift, Method Dispatch]

toc: true
toc_sticky: true

date: 2022-5-14
last_modified_at: 2022-5-14
---

# Method Dispatch 란?

다형성을 제공해주는 객체지향프로그래밍 언어들에서는 하위 클래스에서 상위 클래스의 메소드를 `오버라이드(override)` 해서 사용할 수 있습니다.
`Swift`도 마찬가지로 그런 메서드들을 자주 마주하게되는데, 그런식으로 `override` 된 메서드들이 있으면 컴파일러는 어떤 구현부를 골라야될지 도대체 어떻게 아는걸까요 🤔

이때 어떤 메서드가 호출되어야하는지 결정해주는 과정이 바로 `Method Dispatch`입니다.

즉, `Method Dispatch`란 **어떠한 메서드가 호출되었을때, 어떤 명령어를 실행해야되는지 결정해주는 방법입니다.**

`Method Dispatch` 는 메서드가 호출될때마다 매번 사용되기 때문에, 성능이 좋은 코드를 작성하기 위해서는 `Method Dispatch`의 동작 원리를 어느정도 이해해야될 필요가 있다고 생각해서 이번 포스트에서 다뤄보기로 했습니다.

[<WWDC 2016 Understanding Swift Performance 1부>](https://junbangg.github.io/wwdc/wwdc16-understanding-swift-performance-pt1/#method-dispatch) 에서 `Method Dispatch` 를 잠깐 다뤘어서 같이 정리를 했었는데, 그때 정리했던 글을 다시 보니까 이해 안되는 부분들이 많아서 따로 정리해보는게 좋을것 같았습니다.

이번 포스트에서는 `Method Dispatch` 방법들을 개념적으로 비교해보겠습니다.

# Method Dispatch의 종류

`Dispatch`의 목적은 뭘까요?

`Dispatch` 는 **특정 메서드 호출의 실행 가능한 코드가 메모리 어느 곳에 위치 해있는지 프로그램이 `CPU`에게 알려주는 것** 을 목표로 합니다.

컴파일 언어들에서는 보통 `Dispatch`를 하는 방법이 크게 **Direct Dispatch**, **Indirect Dispatch** 두개 있습니다.

**Direct Dispatch**는 많이 들어봤을만한 정적 디스패치(`Static Dispatch`) 라고도 하며,
**Indirect Dispatch** 가 그 유명한 동적 디스패치(`Dynamic Dispatch`) 입니다.
`Dynamic Dispatch`는 또 두가지 방법으로 나뉘는데, `Table Dispatch`와 `Message Dispatch` 가 있습니다.

정리해보면 `Dispatch` 방법으로 크게 **Static Dispatch**, **Table Dispatch**, **Message Dispatch** 가 있는것입니다.

보통의 컴파일 언어들은 이 3가지 방법 중에 1-2개만 제공하는데, **Swift** 는 굉장한 언어이기 때문에 위 3가지 방법 모두 공부해보라고 전부 제공합니다👍

그럼 하나씩 살펴보겠습니다

## Direct Dispatch (Static Dispatch)



`Static Dispatch` 는 `Method Dispatch` 방법 중 가장 빠른 방법입니다.

`Static Dispatch`로 컴파일러는 **컴파일 타임** 때 바로 어떤 메서드 구현부를 실행해야되는지 알 수 있습니다.
메서드가 호출되면, 컴파일러는 바로 메서드가 위치해있는 메모리 주소로 가서 실행을 시킵니다. 이런식으로 `Static Dispatch`는 호출 과정도 간단할 뿐만 아니라, 어셈블리 명령도 가장 적고 `inlining`처럼 최적화할 수 있는 기법들이 많아진다고 합니다. 

그렇다고 `Static Dispatch`가 장점만 있는건 아닙니다. 성능은 좋지만, **다형성**을 처리할만큼 동적이지 않기 때문에 프로그래밍 측면에서는 제한 사항들이 따라옵니다. 

실제로 `Swift` 에서는 `Static Dispatch`은 모든 **값 타입**에만 적용됩니다. 따라서 **상속** 받은 타입은 `Static Dispatch`으로 `Method Dispatch` 가 불가능합니다.


그렇다면 객체지향프로그래밍의 꽃, **다형성**을 가능하게 해주는 **Dynamic Dispatch**도 한 번 살펴봅시다.

## Indirect Dispatch (Dynamic Dispatch)

<img width="1061" alt="Screen Shot 2022-05-18 at 8 07 30 PM" src="https://user-images.githubusercontent.com/33091784/169025094-e69887f6-f43a-469c-a6e1-709ea3fe2054.png">

([<WWDC 2016 Understanding Swift Performance>](https://developer.apple.com/videos/play/wwdc2016/416/) 슬라이드)

**Dynamic Dispatch** 가 쓰이는 곳은 바로 아래쪽에 있는 `for` 문입니다.

`Drawables` 안에 들어있는 원소의 종류가 `Point` , `Line` 인데 

컴파일러는 **Compile Time** 때 바로 타입을 식별 할 수 없습니다.

즉, `d.draw()` 가 `Point`의 메서드인지 `Line`의 메서드인지 모른다는거죠.

그럼 어떤 implementation을 호출해야되는지 컴파일러는 어떻게 찾을까요??


앞서 이야기한것 처럼, **Dynamic Dispatch** 는 보다 더 유연한 프로그래밍을 가능하게 해줍니다. 

**Dynamic Dispatch**는 특정 인스턴스의 실제 타입에 맞춰서 메서드와 프로퍼티를 호출하기 때문에, 코드상으로는 바로 알 수 있는게 없기 때문에 **런타임** 때 결정을 합니다. 즉, 어떤 서브클래스가 들어와도 실제 타입에 맞는 요소를 참조하기 때문에 **다형성**이 가능해지는거죠.

단, **런타임**때 메서드를 찾는 과정이 있기 때문에 오버헤드가 발생합니다.
즉, **Static Dispatch**와 비교했을때 상대적으로 성능상의 손해가 있습니다.

유연성 vs 성능의 Trade-off 라고 볼 수 있죠

### Table Dispatch vs Message Dispatch

동적 디스패치에는 또 두 가지 방법의 디스패치로 나뉩니다.

#### Table Dispatch
`Table Dispatch` 는 `동적 디스패치`의 가장 기본적인 구현 방식인데, `Table` 을 사용합니다. `Table`이란, 함수 포인터로 이루어진 배열을 의미하며, **V Table**(`Virtual Table`) 이라고 불립니다. (스위프트에서는 `Witness Table` 이라고 합니다.)

특정 클래스의 모든 서브클래스는 각자의 테이블을 갖고있습니다.
이 테이블은 서브클래스가 재정의한(`override`) 모든 메서드에 대해 다른 함수 포인터를 갖고있고, 서브클래스가 새로운 메서드를 추가하면 해당 메소드에 대한 함수 포이터가 테이블(배열) 끝에 추가됩니다.

그래서 **런타임**때 컴파일러는 이 테이블들을 이용해서 올바른 구현을 찾아서 호출하는 것입니다.

**Table Dispatch가 오버헤드인 이유**
컴파일러는 테이블에서 올바른 메서드 구현을 위한 메모리 주소를 **읽고** 그 주소로 **접근**을 해야되기 때문에 2번의 연산이 필요합니다.
즉 메서드가 호출이 될때마다 이런 연사들이 추가로 필요하기 때문에 `Static Dispatch` 보다 느리다고 하는것입니다.


**Message Dispatch**
`Message Dispatch` 는 유연성을 더해줍니다.
클래스가 오버라이드 하거나 새로 정의한 메서드들만 테이블에 유지하고, 부모 타입으로의 포인터를 갖고있어서 부모의 메서드들은 부모 타입으로 찾아가서 실행합니다.

`메세지 디스패치` 이런식으로 유연성을 제공합니다.
`Method Swizziling` 이라는 것을 가능하게 해주는데, 개발자가 디스패치(메서드 구현)를 **런타임에** 동적으로 변경할 수 있게 해주는 것입니다.

단, 이건 `Swift`에서 기본적으로 제공하는것이 아니며, `Objective-C` 런타임에 적용됩니다.
즉, 메서드가 디스패치되면, 런타임떄 클래스 계층을 전부 탐색하면서 메서드를 찾는 꼴이 되며, 성능이 좋지않은것입니다.

한마디로 **가장 유연한 디스패치 방법이자 가장 느린 것입니다**

# 정리




<br>


# 참고 자료

<href>https://www.rightpoint.com/rplabs/switch-method-dispatch-table
<href>https://hyunsikwon.github.io/swift/Swift-MethodDispatch-01/
<href>https://developer.apple.com/swift/blog/?id=27
<href>https://medium.com/@bakshioye/static-vs-dynamic-dispatch-in-swift-a-decisive-choice-cece1e872d
<href>https://developer.apple.com/videos/play/wwdc2016/416/)

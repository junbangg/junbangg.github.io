---
title:  "WWDC 2017 Engineering for Testabillity" 

categories:
  - wwdc
tags:
  - [iOS, wwdc]

toc: true
toc_sticky: true

date: 2022-01-29
last_modified_at: 2022-01-29
---

# 배운 내용
- Testable App Code
    - Protocols and Parameterization
    - Separating logic and effects
- Scalable Testing

# Testable App Code
## Unit Test의 구조
<img width="989" alt="Screen Shot 2022-01-29 at 10 14 27 PM" src="https://user-images.githubusercontent.com/33091784/151662358-4ec11207-83e4-4a1b-9530-1b7f4abb4022.png">


## Characteristics of Testable Code
- Control over Inputs
- Visibility into Outputs
- No hidden state
    - 코드를 바꿀만한 Internal state 를 피한다

## Protocols and Parameterization

<img width="1053" alt="Screen Shot 2022-01-29 at 8 42 05 PM" src="https://user-images.githubusercontent.com/33091784/151660700-d62e8679-c1e7-4c57-86cf-3a9cbaf1241f.png">

이걸 테스트하기 위한 방법

1. UI Test
    단점 1: 실행시간이 길다
    단점 2: `URL` 을 확인할 방법이 없다
2. Unit Test ✅
```swift
func testOpensDocumentURLWhenButtonIsTapped() {
    let controller = UIStoryboard(name: "Main", bundle: nil).instantiateViewController(withIdentifier: "Preview") as! PreviewViewController
    controller.loadViewIfNeeded()
    controller.segmentedControl.selectedSegmentIndex = 1
    controller.document = Document(identifier: "TheID")
    
    controller.openTapped(controller.button)
    ...
}

```

문제점:
- 일단 `ViewController` 안에 있어서 테스트하기가 힘들다.
- 싱글톤의 메서드를 사용하고 있어서 (`UIApplication.shared`) `canOpenURL()` 의 결과를 테스트 코드로 제어할 방법이 없다. 왜냐하면 `Global system state`에 의존하고 있기 때문에 (이 부분 잘 모르겠다)
    - 이걸 호출하면 테스트 앱이 백그라운드로 가고나서 다시 foreground로 불러올 방법이 없다.

이 문제점을 해결하기위해 싱글톤을 내부적으로 사용하지않고 주입을 받으면 된다.

<img width="538" alt="Screen Shot 2022-01-29 at 9 30 43 PM" src="https://user-images.githubusercontent.com/33091784/151661091-bb69e697-d4cc-4022-9a9f-3108e1258368.png">

그리고 `open` 메서드는 다음과 같이 생기게 될거다

<img width="1066" alt="Screen Shot 2022-01-29 at 9 31 11 PM" src="https://user-images.githubusercontent.com/33091784/151661893-ca764786-92a2-4acd-9339-c42ea9e0b7ba.png">


단위 테스트 코드에서는 subclassing, 대신 protocol 사용하는게 좋다

<img width="1198" alt="Screen Shot 2022-01-29 at 9 33 10 PM" src="https://user-images.githubusercontent.com/33091784/151661809-96411278-0953-4e0c-bdf7-64eb1e929989.png">


그래서 `protocol` 을 코드에 적용시켜보면
<img width="1198" alt="Screen Shot 2022-01-29 at 9 35 27 PM" src="https://user-images.githubusercontent.com/33091784/151661860-3fea4fdb-1525-41e0-bf5b-01e1d2bf6b00.png">

<img width="1066" alt="Screen Shot 2022-01-29 at 9 31 11 PM" src="https://user-images.githubusercontent.com/33091784/151661930-aafcf93f-5e4e-4cb0-900d-fca5a96fddd1.png">

이런식으로 생길거다

그러가나서 테스트를 위해 `Mock` 를 사용하는게 좋다. 왜냐하면 `UIApplication` 을 테스트에서 제어할 수가 없다.
그래서 프로토콜에대한 `Mock` 을 다음과 같이 만들 수 있다.


```swift
class MockURLOpener: URLOpening {
    var canOpen = false // input 역할..테스트에서 설정할 수 있게
    var openedURL: URL? // 메서드로 받은 url 저장
    
    func canOpenURL(_ url: URL) -> Bool { // 
        return canOpen
    }
    
    func open( _url: URL, options: [String: Any], completionHandler: ((Bool) -> Void)?) { //
        openedURL = url
    }
}

```

테스트 코드는

```swift
func testDocumentOpenerWhenItCanOpen() {
    let urlOpener = MockURLOpener() 
    urlOpener.canOpen = true 
    let documentOpener = DocumentOpener(urlOpener: urlOpener)
    
    documentOpener.open(Document(identifier: "TheID"), mode: .edit)
    
    XCTAssertEqual(urlOpener.openedURL, URL(string: "alsdkjflkasdjflkjasdlkfj"))
}

```

### 배운것

- Reduce References to shared instances
- Accept parameterized input
    - 의존성 주입
    - 대체성을 위해
- Introduce a protocol
- created a test implementation
    - input 에 대한 제어 제공
    - output에 대한 시야 확보


## Seperating Logic and Effects

이것도 testability를 향상 시킬 수 있는 기법
예시

on-disk cache class
cache된 asset을 불러오는 코드

<img width="1183" alt="Screen Shot 2022-01-29 at 10 05 47 PM" src="https://user-images.githubusercontent.com/33091784/151662143-eb25f7bc-8f6f-4e62-8f63-aab7bc6be1e0.png">

<img width="989" alt="Screen Shot 2022-01-29 at 10 07 53 PM" src="https://user-images.githubusercontent.com/33091784/151662187-48a0ee1a-5b49-48e7-a697-f659f7b4856f.png">

테스트하기 위해서 `input` 과 `output` 를 먼저 생각해볼 수 있는데

<img width="989" alt="Screen Shot 2022-01-29 at 10 15 24 PM" src="https://user-images.githubusercontent.com/33091784/151662391-26914bd6-5f1c-48c4-bf47-70f02d64b174.png">

input 1. cache가 얼만큼 커질 수 있는지에 대한 parameter
input 2. 현재 cache에 저장되어있는 아이템들

ouput: 없다. 즉, 데이터가 안나온다
    - 대신, 파일들이 없어지는 side effect가 발생한다.

그런데 해당 방법은 `FileManager` 에 의존하고 있다는 특징이 있다.

이걸 테스트에서 다뤄야할텐데, 쉽지않다.
전에 배운 protocol, parameterization을 사용해서 테스트에서는, 삭제된 파일의 리스트를 리턴 받아서 validate하는 형식으로도 할 수 있을텐데
그렇게되면, 실제 코드와의 상호작용이 필요하다. 즉, 테스트코드와 실제코드가 독립된 환경이 아니다.


그렇다면 프로토콜 코드를 살펴보면 다음과 같다

<img width="1217" alt="Screen Shot 2022-01-29 at 10 18 23 PM" src="https://user-images.githubusercontent.com/33091784/151662512-5eec9c86-a316-4fb4-ae6f-cfb073cddf0e.png">



<img width="927" alt="Screen Shot 2022-01-29 at 10 20 22 PM" src="https://user-images.githubusercontent.com/33091784/151662580-2a918473-4128-4196-be1e-ba8fff46126a.png">

구현부를 보면, side effect는 없어지고, functional 하게 input과 output이 생겼다.

테스트 코드는 다음과 같다

<img width="927" alt="Screen Shot 2022-01-29 at 10 49 05 PM" src="https://user-images.githubusercontent.com/33091784/151663514-c07b8b8e-92ca-47a5-b9ce-014fb17df90f.png">

- input에 대한 제어 + output에 대한 시야 + hidden state가 없다
- FileManager 에 의존을 안해도 돼서 빠르다

실제 코드도 이렇게 바뀐다
<img width="927" alt="Screen Shot 2022-01-29 at 10 54 32 PM" src="https://user-images.githubusercontent.com/33091784/151663694-4236018f-cd25-44e6-9d74-1097160a1012.png">

훨씬 직관적이고 테스트성이 좋아졌다


### 배운것
- extract algorithms
    - business logic & algorithms를 각각 다른 타입으로 분리
- functional style with value types
- thin layer on top to execute effects
  - 남아있는 코드 

# Scalable Testing


- faster
- readable
- modularized

## 주제
- Balance between UI and Unit Tests
- Code to help UI tests scale
- Quality of test code

## Balance between UI and Unit Tests

<img width="927" alt="Screen Shot 2022-01-29 at 10 59 42 PM" src="https://user-images.githubusercontent.com/33091784/151663882-593d4993-eb37-411e-bf15-c3857aa754b2.png">

아래로 갈 수록 Distribution은 커지고, Maintenance Cost 는 낮아진다

왜냐하면 UI Test 보다 Unit Test 가 훨신 빨라서 많아도 되고
테스트에서 뭐가 잘못되면 UI Test 보다 Unit Test에서 더 캐치하기 쉬워서 Maintenance Cost 도 더 낮다고한다. 반면에 UI Test 는 어디서 뭐가 틀렸는지 캐치하기 더 어렵다

<img width="785" alt="Screen Shot 2022-01-29 at 11 03 50 PM" src="https://user-images.githubusercontent.com/33091784/151664004-9861cade-0cdb-4d29-bf88-10379e383bfc.png">

추가로, Unit Test는 모든 소스코드에 접근이 가능한 반면에 UI Test는 아니다.

## Writing Code to help UI tests scale

<img width="785" alt="Screen Shot 2022-01-29 at 11 05 58 PM" src="https://user-images.githubusercontent.com/33091784/151664073-e347cc0a-f2b7-4dbf-8033-532db2a0f8ee.png">



### Abstracting UI element queries
- Store parts of queries in variable
- wrap complex queries in utility methods
- reduces noise and clutter in UI test

<img width="785" alt="Screen Shot 2022-01-29 at 11 16 31 PM" src="https://user-images.githubusercontent.com/33091784/151664344-e74bd6a7-0b24-4e38-b1df-9affc3f35833.png">

이렇게 하는것보다

<img width="855" alt="Screen Shot 2022-01-29 at 11 17 08 PM" src="https://user-images.githubusercontent.com/33091784/151664369-1cf8c29c-25a1-4a4a-b093-cd022cda4613.png">

이렇게하면 추후에 버튼을 추가할때 그냥 배열에만 넣어주면 된다.!


## Creating Objects and utility functions

<img width="855" alt="Screen Shot 2022-01-29 at 11 21 36 PM" src="https://user-images.githubusercontent.com/33091784/151664499-39d0d2c6-1a04-414d-abe4-078dcbc9c5ad.png">

이건 Scalable 하지 않은 코드다


<img width="514" alt="Screen Shot 2022-01-29 at 11 23 48 PM" src="https://user-images.githubusercontent.com/33091784/151664567-17f444ee-485d-49da-955a-5a63319368b9.png">
이렇게 바꿔도 되고

<img width="542" alt="Screen Shot 2022-01-29 at 11 24 31 PM" src="https://user-images.githubusercontent.com/33091784/151664579-d6259eaa-c2c2-4d44-9f70-215dc6114d33.png">

`enum` 을 사용하면 컴파일 되기도 전에 파라미터로 들어오는 인자를 확인할 수 있다




<img width="748" alt="Screen Shot 2022-01-29 at 11 26 04 PM" src="https://user-images.githubusercontent.com/33091784/151664627-9a907ed8-38d0-453a-b7f1-3acbe891ddd6.png">

그럼 이렇게 코드가 진화한다


<img width="989" alt="Screen Shot 2022-01-29 at 11 26 41 PM" src="https://user-images.githubusercontent.com/33091784/151664651-43038e86-374e-4b77-85d2-4642fbaa5160.png">

이 부분도 다음과 같이 보완할 수 있다


<img width="1001" alt="Screen Shot 2022-01-29 at 11 28 42 PM" src="https://user-images.githubusercontent.com/33091784/151664741-f901e24e-27a2-49ff-ba64-034a61ff30e8.png">

<img width="1001" alt="Screen Shot 2022-01-29 at 11 29 10 PM" src="https://user-images.githubusercontent.com/33091784/151664760-cd7527be-6ffb-4881-a4fc-df8cda25947a.png">


<img width="778" alt="Screen Shot 2022-01-29 at 11 30 11 PM" src="https://user-images.githubusercontent.com/33091784/151664796-861b6345-3a89-4fc7-8fc1-9eb54dc6db8c.png">

<img width="1057" alt="Screen Shot 2022-01-29 at 11 31 42 PM" src="https://user-images.githubusercontent.com/33091784/151664848-6961d891-43a4-4bf8-8488-b5d80a8f7f87.png">

`XCTContent.runActivity` 활용하면 테스트 log 를 깔끔하게 할 수 있다


<img width="1057" alt="Screen Shot 2022-01-29 at 11 35 23 PM" src="https://user-images.githubusercontent.com/33091784/151664994-d2b2ea5f-1b2d-4746-b10b-a0ff23237b77.png">

테스트 코드도 실제 코드처럼 신경써줘야 함께 확장이 가능하다!

---
toc: true
toc_sticky: true
title: "Notification Center"
exerpt: "NotificationCenter"
categories: iOS
tags: iOS
date: 2021-12-3 15:00:00
---

## NotificationCenter

특정 `Event` 가 발생했을때 `NotificationCenter` 한테 `post` 를 이용해서 `Notification` 을 전달하는것을 배웠었다.

그렇게되면 `NotificationCenter` 에 등록이 되어있던 `Observer` 가 `Notification` 에 대한 처리를 한다.

그런데 만약에 `Notification` 을 통해 특정 정보를 전달해야될때는 어떻게 해야되는지 공부해봤다.

이걸 구현하기 위해서는

`post` 함수랑 `addObserver` 의 `selector` 메서드를 조금씩 바꿔줘야한다.

`post` 함수에서

```swift
func post(name aName: NSNotification.Name, 
   object anObject: Any?, 
 userInfo aUserInfo: [AnyHashable : Any]? = nil)
```

위와 같이 `userInfo` 를 통해 `NotificationCenter` 로 데이터를 전달하면 되는데,

`userInfo` 는 `[AnyHashable : Any]?` 의 타입으로 저장을하면 된다.

즉, 전달하고싶은 데이터를 

`key` 와 `value` 로 전달해서 

`addObserver` 의 `#selector` 메서드로 받아서 적절히 적용시키면 된다.

```swift
NotificationCenter.default.addObserver(self,
                                  selector: #selector(newEvent),
                                  name: Notification.Name.newEvent,
                                  object: nil)
```

단, `addObserver` 에서 `newEvent` `#selector` 메서드를 만들때

```swift
@objc private func newEvent(notification: Notification) {
```

이런식으로 `Notification` 을 매개변수로 받도록 구현해야된다.

근데 여기서 특이한게, 위에 `addObserver` 코드에서 `selector: #selector(newEvent)` 를 할때 `newEvent` 에 매개변수를 안넣는다. 

이게 왜 이런건지 좀 궁금하다.

`Notification` 을 저기다 넣는 방법이 없긴한데, 매개변수 없이 `Notification` 이 어떻게 메서드 안으로 전달되는 것인지 궁금하다.
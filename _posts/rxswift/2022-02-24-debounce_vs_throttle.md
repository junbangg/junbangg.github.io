---
title:  "[RxSwift 뜯어보기]Debounce vs Throttle" 

categories:
  - rxswift
tags:
  - [iOS, rxswift]

toc: true
toc_sticky: true

date: 2022-02-24
last_modified_at: 2022-02-24
---
# Debounce vs Throttle

오늘은 Debounce 와 Throttle의 차이점을 알아봐야겠다
둘이 은근 얼핏봤을때 비슷한것 같아서 한 번 깊게 다루면 기억에 남을것 같다

# debounce
Rx 공식문서에 의하면 [debounce](https://reactivex.io/documentation/operators/debounce.html) 는
```
only emit an item from an Observable if a particular timespan has passed without it emitting another item
```
라고 한다.
특징은 다음과 같다.
- **event 를 방출한 다음 지정해둔 시간동안 event를 방출하지않는다면, 해당 시점에 가장 마지막에 전달된 이벤트를 구독자에게 전달한다고 볼 수 있다**
- 그리고 만약에 지정된 시간 이내에 event가 방출됐다면, timer 는 초기화된다.

debounce 는 `combine` 으로 한번 접해본 적이 있는데 그때 검색 기능 구현할때 사용했었다
```swift
$searchQuery
            .debounce(for: .milliseconds(750), scheduler: RunLoop.main)
```
진짜 그냥 똑같이 생겼다..이걸 사용한 이유는 
문자가 입력될때마다 매번 네트워크 호출을 실행시키는게 효율적이지 않아서 짧은 시간동안 연속해서 문자 입력할때는 debounce 가 검색(작업)을 막아줬었다. 그다음에 지정된 시간동안 입력이 없으면, 그때서야 검색을 할 수 있도록 구현한거다. 이렇게 되면 불필요한 리소스를 낭비할 필요 없이 실시간 검색 기능 구현이 가능하다.

## debounce 뜯어보기
```swift
extension ObservableType {

    /**
     Ignores elements from an observable sequence which are followed by another element within a specified relative time duration, using the specified scheduler to run throttling timers.

     - seealso: [debounce operator on reactivex.io](http://reactivex.io/documentation/operators/debounce.html)

     - parameter dueTime: Throttling duration for each element.
     - parameter scheduler: Scheduler to run the throttle timers on.
     - returns: The throttled sequence.
     */
    public func debounce(_ dueTime: RxTimeInterval, scheduler: SchedulerType)
        -> Observable<Element> {
            return Debounce(source: self.asObservable(), dueTime: dueTime, scheduler: scheduler)
    }
}
```
일단 파라미터로 `dueTime` (지정한 시간), 그리고 `scheduler` 를 받고
`RxTimeInterval` 은 왠지 열거형일거 같은데
들어가보니까 `DispatchTimeInterval` 열거형을 한 번 감싼 `typealias` 이다

아무튼 그래서 걔네를 이용해서 `Debounce` 라는 타입을 생성해서 리턴하는데 그럼 `Debounce` 는 뭔지 한 번 보러가자

```swift

final private class Debounce<Element>: Producer<Element> {
    fileprivate let source: Observable<Element>
    fileprivate let dueTime: RxTimeInterval
    fileprivate let scheduler: SchedulerType

    init(source: Observable<Element>, dueTime: RxTimeInterval, scheduler: SchedulerType) {
        self.source = source
        self.dueTime = dueTime
        self.scheduler = scheduler
    }

    override func run<Observer: ObserverType>(_ observer: Observer, cancel: Cancelable) -> (sink: Disposable, subscription: Disposable) where Observer.Element == Element {
        let sink = DebounceSink(parent: self, observer: observer, cancel: cancel)
        let subscription = sink.run()
        return (sink: sink, subscription: subscription)
    }
    
}
```
`Debounce` 는 `Producer` 라는 클래스를 상속받은 타입인데, 별다른건 없고 `Producer` 에는 `run` 이라는 메서드가 있는데 그걸 `override` 해서 사용하나보다
`run` 은 `DebounceSink`랑 `DebounceSink().run()` 의 리턴값을 튜플 형태로 반환하는데 그렇다면 `DebounceSink` 가 뭔지도 한 번 살펴봐야겠다

```swift
final private class DebounceSink<Observer: ObserverType>
    : Sink<Observer>
    , ObserverType
    , LockOwnerType
    , SynchronizedOnType {
    typealias Element = Observer.Element 
    typealias ParentType = Debounce<Element>

    private let parent: ParentType

    let lock = RecursiveLock()

    // state
    private var id = 0 as UInt64
    private var value: Element?

    let cancellable = SerialDisposable()

    init(parent: ParentType, observer: Observer, cancel: Cancelable) {
        self.parent = parent

        super.init(observer: observer, cancel: cancel)
    }

    func run() -> Disposable {
        let subscription = self.parent.source.subscribe(self)

        return Disposables.create(subscription, cancellable)
    }

    func on(_ event: Event<Element>) {
        self.synchronizedOn(event)
    }

    func synchronized_on(_ event: Event<Element>) {
        switch event {
        case .next(let element):
            self.id = self.id &+ 1
            let currentId = self.id
            self.value = element


            let scheduler = self.parent.scheduler
            let dueTime = self.parent.dueTime

            let d = SingleAssignmentDisposable()
            self.cancellable.disposable = d
            d.setDisposable(scheduler.scheduleRelative(currentId, dueTime: dueTime, action: self.propagate))
        case .error:
            self.value = nil
            self.forwardOn(event)
            self.dispose()
        case .completed:
            if let value = self.value {
                self.value = nil
                self.forwardOn(.next(value))
            }
            self.forwardOn(.completed)
            self.dispose()
        }
    }

    func propagate(_ currentId: UInt64) -> Disposable {
        self.lock.performLocked {
            let originalValue = self.value

            if let value = originalValue, self.id == currentId {
                self.value = nil
                self.forwardOn(.next(value))
            }

            return Disposables.create()
        }
    }
}

```

코드가 꽤 복잡하고 어려워보여서 일단은 `run` 이 뭐하는지부터 살펴봐야겠다.
```swift
func run() -> Disposable {
        let subscription = self.parent.source.subscribe(self)

        return Disposables.create(subscription, cancellable)
    }
```
여기서 `parent.source`, 즉 `debounce` 를 시킨 대상이 될것 같은데 그 친구에게 이 `DebounceSink` 를 구독시켜서 `Disposable` 을 만들어서 리턴한다
그럼 지금까지 살펴본 코드로만 봤을때 `.debounce()` 의 기본적인 동작은 그냥 `Observable` 로 감싼 타입이 하나 리턴되고, 그거에 `subscribe` 를 하는 형태인것 같다.

사실 이걸 분석해보고싶었던 이유는, `debounce` 할때 지정해둔 시간(`DispatchTimeInterval`) 이 어떻게 사용되는지 보고싶었다.
파보면 결국엔 `asyncAfter` 같은 코드가 나오지않을까라는 생각이 들었는데 그런게 없어서 좀 신기하다
정확히 저 시간이라는 개념이 어떻게 작동하는지는 `func synchronized_on(_ event: Event<Element>)` 여기에 는것 같다.

```swift
    func synchronized_on(_ event: Event<Element>) {
        switch event {
        case .next(let element):
            self.id = self.id &+ 1
            let currentId = self.id
            self.value = element


            let scheduler = self.parent.scheduler
            let dueTime = self.parent.dueTime

            let d = SingleAssignmentDisposable()
            self.cancellable.disposable = d
            d.setDisposable(scheduler.scheduleRelative(currentId, dueTime: dueTime, action: self.propagate))
        case .error:
            self.value = nil
            self.forwardOn(event)
            self.dispose()
        case .completed:
            if let value = self.value {
                self.value = nil
                self.forwardOn(.next(value))
            }
            self.forwardOn(.completed)
            self.dispose()
        }
    }
```

이 코드를 보면 `event` 를 파라미터로 받고, `switch` 문으로 `.next, .error, .completed` 를 구분해서 동작을 취한다.

`next` 케이스를 보면, 
`SingleAssignmentDisposable` 에 `scheduler.scheduleRelative` 를 넣는데, 여기서 그토록 궁금했던 `dueTime` 이 들어간다.
그럼 결국 일을 하는 친구는 이 `scheduler` 인것 같다
따라 들어가보니까
```swift

    /**
    Schedules an action to be executed.
    
    - parameter state: State passed to the action to be executed.
    - parameter dueTime: Relative time after which to execute the action.
    - parameter action: Action to be executed.
    - returns: The disposable object used to cancel the scheduled action (best effort).
    */
    public final func scheduleRelative<StateType>(_ state: StateType, dueTime: RxTimeInterval, action: @escaping (StateType) -> Disposable) -> Disposable {
        self.configuration.scheduleRelative(state, dueTime: dueTime, action: action)
    }
```
이런 코드가 있는데, 여기서 `scheduleRelative` 도 들어가보면
```swift

    func scheduleRelative<StateType>(_ state: StateType, dueTime: RxTimeInterval, action: @escaping (StateType) -> Disposable) -> Disposable {
        let deadline = DispatchTime.now() + dueTime

        let compositeDisposable = CompositeDisposable()

        let timer = DispatchSource.makeTimerSource(queue: self.queue)
        timer.schedule(deadline: deadline, leeway: self.leeway)

        // TODO:
        // This looks horrible, and yes, it is.
        // It looks like Apple has made a conceputal change here, and I'm unsure why.
        // Need more info on this.
        // It looks like just setting timer to fire and not holding a reference to it
        // until deadline causes timer cancellation.
        var timerReference: DispatchSourceTimer? = timer
        let cancelTimer = Disposables.create {
            timerReference?.cancel()
            timerReference = nil
        }

        timer.setEventHandler(handler: {
            if compositeDisposable.isDisposed {
                return
            }
            _ = compositeDisposable.insert(action(state))
            cancelTimer.dispose()
        })
        timer.resume()

        _ = compositeDisposable.insert(cancelTimer)

        return compositeDisposable
    }
```
이런 코드가 나온다. 11번째줄에 주목해볼만한것 같다. "This looks horrible" 이라고 써있는데
공감이 꽤 되는 말이다.

진짜 어떻게 동작하는지 모르겠다 이건
`throttle` 도 비슷할것 같은데 한 번 가보자

# throttle

`throttle` 는 공식문서에서 글을 못찾았다
대신 구현되어있는 코드 주것을 살펴보니까
```
Returns an Observable that emits the first and the latest item emitted by the source 
Observable during sequential time windows of a specified duration.
```
이렇게 써있었다
`debounce` 와는 다르게 "**next 이벤트를 지정된 주기마다 하나씩 구독자에게 전달한다**" 로 정리할 수 있을것 같다.
근데 코드를 살펴보니까 `debounce`랑 비슷한 형태로 구현되어있다


```swift

extension ObservableType {

    /**
     Returns an Observable that emits the first and the latest item emitted by the source Observable during sequential time windows of a specified duration.

     This operator makes sure that no two elements are emitted in less then dueTime.

     - seealso: [debounce operator on reactivex.io](http://reactivex.io/documentation/operators/debounce.html)

     - parameter dueTime: Throttling duration for each element.
     - parameter latest: Should latest element received in a dueTime wide time window since last element emission be emitted.
     - parameter scheduler: Scheduler to run the throttle timers on.
     - returns: The throttled sequence.
     */
    public func throttle(_ dueTime: RxTimeInterval, latest: Bool = true, scheduler: SchedulerType)
        -> Observable<Element> {
        Throttle(source: self.asObservable(), dueTime: dueTime, latest: latest, scheduler: scheduler)
    }
}
final private class Throttle<Element>: Producer<Element> {
    fileprivate let source: Observable<Element>
    fileprivate let dueTime: RxTimeInterval
    fileprivate let latest: Bool
    fileprivate let scheduler: SchedulerType

    init(source: Observable<Element>, dueTime: RxTimeInterval, latest: Bool, scheduler: SchedulerType) {
        self.source = source
        self.dueTime = dueTime
        self.latest = latest
        self.scheduler = scheduler
    }
    
    override func run<Observer: ObserverType>(_ observer: Observer, cancel: Cancelable) -> (sink: Disposable, subscription: Disposable) where Observer.Element == Element {
        let sink = ThrottleSink(parent: self, observer: observer, cancel: cancel)
        let subscription = sink.run()
        return (sink: sink, subscription: subscription)
    }
    
}
```
파라미터도 거의 똑같고(`latest:Bool` 이 있는것 빼고)
`Observable`를 한 번 깜싸서 `Throttle`의 형태로 반환하는것도 똑같은걸 볼 수 있다

`debounce`와의 차이는 저 `DebounceSink` 과 `ThrottleSink` 인것 같은데 저걸 살펴보면 좀 알 수 있지않을까 싶다
```swift
func synchronized_on(_ event: Event<Element>) {
        switch event {
        case .next(let element):
            let now = self.parent.scheduler.now

            let reducedScheduledTime: RxTimeInterval

            if let lastSendingTime = self.lastSentTime {
                reducedScheduledTime = self.parent.dueTime.reduceWithSpanBetween(earlierDate: lastSendingTime, laterDate: now)
            }
            else {
                reducedScheduledTime = .nanoseconds(0)
            }

            if reducedScheduledTime.isNow {
                self.sendNow(element: element)
                return
            }

            if !self.parent.latest {
                return
            }

            let isThereAlreadyInFlightRequest = self.lastUnsentElement != nil
            
            self.lastUnsentElement = element

            if isThereAlreadyInFlightRequest {
                return
            }

            let scheduler = self.parent.scheduler

            let d = SingleAssignmentDisposable()
            self.cancellable.disposable = d

            d.setDisposable(scheduler.scheduleRelative(0, dueTime: reducedScheduledTime, action: self.propagate))
        case .error:
            self.lastUnsentElement = nil
            self.forwardOn(event)
            self.dispose()
        case .completed:
            if self.lastUnsentElement != nil {
                self.completed = true
            }
            else {
                self.forwardOn(.completed)
                self.dispose()
            }
        }
    }
```
결국 `debounce` 와 `throttle` 의 차이점은, `DebounceSink/ThrottleSink`에 구현되어있는 저 `synchronized_on` 메서드인것 같다.
`throttle` 이 훨씬 복잡하고 읽기 싫게 생긴게 특징이다.

# debounce vs throttle

정말 간단하고 대충 두 `operator` 의 차이점을 코드로 살펴봤는데 
```swift=
// TODO:
        // This looks horrible, and yes, it is.
```
이게 맞는것 같다

둘다 `synchronized_on` 메서드에서 `scheduler` 를 적절히 제어해서 동작한는것 같은데
그 이상은 이해하기 어려웠다.

공부를 더 해보면서 추후에 이 글도 좀 손을 봐야겠다. 특히, `synchronized_on` 이 정확히 어떻게 동작하는지 한 번 자세히 봐야겠다.

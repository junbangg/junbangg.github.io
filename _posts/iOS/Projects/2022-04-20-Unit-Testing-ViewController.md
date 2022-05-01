---
title: "ViewController를 단위테스트 하는 방법(feat. POP)"

categories:
  - Projects
tags:
  - [iOS, Projects]

toc: true
toc_sticky: true

date: 2022-04-20
last_modified_at: 2022-04-20
---
# 뷰 컨트롤러 테스트하는 방법

오늘은 `ViewController` 를 테스트하기 위한 여러가지 방법 중 **단위 테스트** 를 알아보고 직접 프로젝트에 적용해본 경험을 기록해보려고합니다.

`View` 를 테스트하기 위한 방법으로 `Snapshot` 테스트를 활용한 `UI 테스트` 도 있지만,

본 포스팅에서는 `단위 테스트` 가 뭔지, 그리고 `단위 테스트` 로 테스트할만한 이유들 부터 한 번 정리해보겠습니다.

## Unit Test란?

- 기능/단위 별로 (함수와 메서드) 테스트 케이스를 작성하는 절차
- 언제라도 코드 변경으로 인해 문제가 발생할 경우, 단시간 내에 이를 파악하고 바로 잡을 수있도록 해준다.

## Unit Test 의 장점

- 작성하기 비교적 쉽고 빠르다
- 프로그램의 각 부분을 고립시켜서, 정확하게 동작하는지 확인할 수 있다
    - 이러면 문제가 발생했을때도 어느 부분이 잘못되었는지 확인할 수 있다.
- 유지보수하기 쉽다
    - 좋은 유닛 테스트 디자인은 그 유닛이 사용되는 모든 경로를 커버할 수 있는 테스트 케이스를 만들어 준다.
- 테스트 대상의 코드를 직접적으로 테스트하는것이기 때문에 디버깅하기 쉽다.
- 통합이 간단하다

위와 같은 이유들 때문에, 레이아웃에 대한 테스트 처럼 꼭 `UI Test` 를 사용해야되는 이유가 아니라면 `단위 테스트` 를 고려해봐도 되겠다는 생각을 했습니다.

# 그럼 `ViewController` 를 어떻게 테스트할까

`ViewController` 테스트의 핵심은 `의존성 주입` (`Dependency Injection`) 이라고 생각합니다. 

[의존성 주입 포스팅]([https://junbangg.github.io/swift/dependency-injection/](https://junbangg.github.io/swift/dependency-injection/)) 에도 정리를 해놨었는데, 

**클래스 간의 결합도를 낮추고, 테스트성을 높이는** 디자인 패턴입니다.

(그 외에도 **의존성 주입** 이 가져다주는 이점들은 많지만, 본 포스팅에서는 테스트성과 관련해서만 기록을 해보겠습니다.)

**의존성 주입** 을 사용하면 뷰컨트롤러가 가지고있는 의존 관계들을 전부 역전 시킬 수 있고, 해당 의존 타입들 대신 **테스트 더블** 들을 주입 시켜 테스트를 할 수 있게 됩니다.

# 개인 프로젝트 `Stocky` 에 적용해보기

## 회고

공부해본 내용을 제 개인프로젝트인  [Stocky](https://github.com/junbangg/Stocky)에 적용해봤습니다.

`Stocky` 의 계산기 뷰에서는 테스트를 면밀히 해야되는 기능이 크게 두가지가 있습니다.

- 투자 종목의 수익성 유무를 계산하는 기능
- 수익성의 유무에 따라 `UI` 의 상태를 결정하는 기능

![](https://i.imgur.com/Z8tgSNV.gif)


만약에 위 뷰를 책임지고있는 뷰컨트롤러 `CalculatorViewController` 가 있다고 가정해보겠습니다.

그리고 위에 언급했던 두 가지 주요 로직을 담당하는 메서드와, 관련된 타입들이 전부 이 뷰컨트롤러 안에 있다고 하겠습니다.

## 타입 소개

### `DCAResult`

매입원가 평균법으로 투자종목의 수익성 정보를 담은 타입

```swift
struct DCAResult {
    let currentValue: Double
    let investmentAmount: Double
    let gain: Double
    let yield: Double
    let annualReturn: Double
    let isProfitable: Bool
}
```

### `calculateDCAResult()`

투자 종목의 정보를 이용해서 매입원가 평균법으로 수익성을 계산하는 메서드

```swift
func calculateDCAResult(
	_ asset: Asset,
  _ initialInvestmentAmount: Double,
  _ monthlyDollarCostAveragingAmount: Double,
  _ initialDateOfInvestmentIndex: Int
) -> DCAResult
```

### `CalculatorUIPresentation`

UI 에 최종적으로 나타낼 문자열/컬러 를 담은 타입

```swift
struct CalculatorUIPresentation {
    let currentValueLabelBackgroundColor: UIColor
    let currentValue: String
    let investmentAmount: String
    let gain: String
    let yield: String
    let yieldLabelTextColor: UIColor
    let annualReturn: String
    let annualReturnLabelTextColor: UIColor
}
```

### `getPresentation()`

`DCAResult` 를 이용해서 `CalculatorUIPresentation` 인스턴스를 만들어서 반환하는 메서드

```swift
func getPresentation(result: DCAResult) -> CalculatorUIPresentation
```

## 근데 그래서 테스트를 어떻게 할까?🤔

위 처럼 뷰컨트롤러가 해당 메서드들을 그냥 자체적으로 갖고 있으면 뷰컨트롤러를 통째로 `mock` 해서 테스트 해야됩니다.

물론 그게 필요한 상황도 충분히 있을 수 있겠지만, 위의 로직에 대한 테스트를 하기 위해서는 각 기능을 분리하는게 좋다고 판단했습니다.

또한, (어쩌면 아키텍쳐 관련된 이야기가 될 수도 있지만), 본 앱은 현재 `MVC` 아키텍쳐로 설계되어 있기 때문에 다음과 같은 문제점이 생길 수 있다고 생각했습니다. 

- 뷰컨트롤러에 변경 사항이 생기면 위 로직들도 변경을 해야되는 상황이 발생할 수 있다
- 위 로직에 변경 사항이 생기면 뷰컨트롤러 통째로 변경이 되어야하는 상황이 발생할 수 있다.

그렇다면 `MVC` 에서 뷰컨트롤러를 효과적으로 테스트 하기 위해서는 어떻게 해야될지 고민을 해보다가, `POP` 를 이용해보기로 했습니다.

## POP 로 (살짝) 리팩토링

`POP` 에 대해서 [WWDC 2016년도 Protocol and Value Oriented Programming in UIKit Apps](https://developer.apple.com/videos/play/wwdc2016/419/) 를 보면서 [공부 해보고 기록을 해뒀었는데](https://junbangg.github.io/wwdc/wwdc16-protocol-and-value-oriented-programming-in-uikit-apps/), 테스트할때도 효과가 있다는게 얼핏 기억나서 한 번 적용해봤습니다.

위에 테스트하기로 했던 두개의 기능들

- 투자 종목의 수익성 유무를 계산하는 기능
- 수익성의 유무에 따라 `UI` 의 상태를 결정하는 기능

이 두개의 기능을 **프로토콜** 로 다음과 같이 두개의 프로토콜로 분리해보기로 했습니다.

- `DCAServicable`
- `CalculatorUIPresentable`

이렇게 하고 확장을 이용해서 각 메서드의 기본 구현을 제공했다. 

```swift
protocol DCAServicable {
    func calculateDCAResult(
        _ asset: Asset,
        _ initialInvestmentAmount: Double,
        _ monthlyDollarCostAveragingAmount: Double,
        _ initialDateOfInvestmentIndex: Int
    ) -> DCAResult
}

// MARK: - DCAServicable Methods

extension DCAServicable {
    func calculateDCAResult(
        _ asset: Asset,
        _ initialInvestmentAmount: Double,
        _ monthlyDollarCostAveragingAmount: Double,
        _ initialDateOfInvestmentIndex: Int
    ) -> DCAResult {
			// 구현부       
    }
}
```

```swift
protocol CalculatorUIPresentable {
    func getPresentation(result: DCAResult) -> CalculatorUIPresentation
}

// MARK: - CalculatorUIPresentable Methods

extension CalculatorUIPresentable {
    func getPresentation(result: DCAResult) -> CalculatorUIPresentation {
			// 구현부
    }
}
```

이렇게 구현을 한뒤에 해당 뷰컨트롤러에 채택 시키면 기능은 분리 시키면서 사용할 수 있다.

```swift
final class CalculatorTableViewController: UITableViewController {}

// MARK: - CalculatorUIPresentable

extension CalculatorTableViewController: CalculatorUIPresentable {}

// MARK: - DCAServicable

extension CalculatorTableViewController: DCAServicable {}
```

## 근데 그래서 테스트를 어떻게 할까😅

이제 드디어 테스트를 할 준비를 마쳤습니다.

위의 두 프로토콜을 뷰컨트롤러와 분리된 상태로 개별적으로 테스트하면 됩니다.

### `DCAServicable` 테스트

`MockDCAService` 구조체를 하나 만들어놓고 `DCAServicable` 프로토콜을 뷰컨트롤러한테 했듯이 달아놓으면 됩니다.

```swift
import Foundation
@testable import Stocky

struct MockDCAService: DCAServicable {
    
}
```

그리고 테스팅을 하면 됩니다.

```swift
final class DCAServiceTests: XCTestCase {
    private var sut: DCAServicable!
    
    override func setUpWithError() throws {
        try super.setUpWithError()
        sut = MockDCAService()
    }
    
    override func tearDownWithError() throws {
        try super.tearDownWithError()
        sut = nil
    }
	// 즐거운 테스팅:)
}
```

### `CalculatorUIPresentable` 테스트

`CalculatorUIPresentable` 도 마찬가지로 `mock` 에 붙여서 테스트 하면 됩니다.

```swift
struct MockCalculatorUIPresenter: CalculatorUIPresentable {
    
}
```

```swift
final class CalculatorPresenterTests: XCTestCase {
    var sut: CalculatorUIPresentable!

    override func setUpWithError() throws {
        sut = MockCalculatorUIPresenter()
        try super.setUpWithError()
    }
    
    override func tearDownWithError() throws {
        sut = nil
        try super.tearDownWithError()
    }
		//즐테 :)
}
```

# 마무리

`MVC` 아키텍쳐로 설계를 한 앱에서 뷰컨트롤러 로직을 테스트하는 방법에 대해서 고민해본 과정을 기록 해봤습니다.

요즘 클린 아키텍쳐에 관심이 생겨서 본 프로젝트를 한 번 리팩토링 해볼 예정인데, 위 코드가 얼마나 영향을 받을지에 대해서 한 번 더 기록해보겠습니다:)

# 참고 자료

[https://www.vadimbulavin.com/unit-testing-view-controller-uiviewcontroller-and-uiview-in-swift/](https://www.vadimbulavin.com/unit-testing-view-controller-uiviewcontroller-and-uiview-in-swift/)

[https://www.objc.io/issues/1-view-controllers/testing-view-controllers/](https://www.objc.io/issues/1-view-controllers/testing-view-controllers/)

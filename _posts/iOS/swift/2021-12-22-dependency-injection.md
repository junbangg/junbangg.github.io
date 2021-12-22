# 의존성 주입

외부에서 두 객체 간의 관계를 결정해주는 디자인 패턴으로

특정 객체가 필요로하는 것들을 내부적으로 생성하게 두지 않고, 외부에서 제공해주는 것.

```swift
class Macbook {
	private let ram = RAM()
	private let processor = M1()
	private let os = MacOS()
}
```

위와 같은 경우는 `Macbook` 클래스가 자신이 필요로하는 의존성 객체(dependency object)에 대한 참조를 직접 만들고있는 것이다.

이러면 클래스간의 **결합도가 높아지고**, 자연스럽게 **테스트성도 낮아진다.**

우리가 좋아하는 `Macbook` 을 생각해보자.

한 번 사면 바꿀 수 있는게 **거의** 없다. (물론 프로세서를 바꿔야되는 이유가 없을지도 모르겠지만)

아무튼, 한 번 구매하면 내용물을 바꾸기가 굉장히 힘들어진다.

이걸 근데 한 번 **의존성 주입** 방법으로 바꿔보기 위해 조립식 PC로 객체를 바꿔보자.

```swift
class 조립식PC {
	private let ram: Ram
	private let processor: M1
	private let os: MacOS
	
	// 생성자 방법
	init(ram: Ram, processor: M1, os: MacOS) {
		self.ram = ram
		self.processor = processor
		self.os = os
	}
	
	//setter 방법
	func setRam(with ram: Ram) {
		self.ram = ram
	}

	func setProcessor(with processor: M1) {
		self.processor = processor
	}

	func setOS(with os: MacOS) {
		self.os = os
	}
}
```

이렇게 **생성자 방법**, 또는, **setter** 를 이용한 방법으로 고쳐볼 수 있다.

(의존성주입 하는 방법도 여러가지가 있는데 그건 밑에서 다시 한 번 살펴볼 것이다.)

이렇게 **의존성 주입** 으로 바꿔보면 생기는 이점은 뭐가 있을까?

얼핏보면 코드만 길어진것 같은데 말이다.

## 컴파일 타임에서 런타임으로

**의존성 주입**을 하게되면 객체가 의존성을 받는 시점을 **컴파일타임**이 아닌 **런타임**으로 늦춰줄 수 있다.

그럼 클래스가 생성된 이후에도 변경사항을 계속 해줄 수 있는 **유연성**이 생긴다.

위의 `Macbook` 클래스에서 `ram` 을 만약에 추가하고싶다고 가정해보자.

한 번 만들어진 `Macbook` (**컴파일 타임**) 은 내용물을 바꾸기가 쉽지않다는 것은 누구나 알것이다.

방법이 뭐가 있을까...

뒤에 뜯어서 고치고싶어도..요즘 맥들은 ram이 motherboard 와 **결합**되어있는 경우가 있어서

사용자가 직접 고치는것은 쉽지 만은 않은 일이다.

`Macbook` 이라는 객체와 그 안에 있는 구성요소들의 **결합도** 가 너무 높아서 생기는 문제점이라고 볼 수도 있을것 같다.

그런데 만약에 컴퓨터라는 객체의 구성요소들을 만드는 시점을 **런타임** 때로 늦춘다면 어떨까?

그러면 구성요소들을 바꾸고싶으면 언제든지 바꿀 수 있는 **유연성**을 제공할 수 있을것 같다.

다시말해서, 객체를 직접적으로 변경하지 않으면서 dependencies 들을 바꿔줄 수 있는 **유연성** 과 **확장성** 을 제공한다고 볼 수 있다.

## 객체간의 결합도를 낮출 수 있다.

객체간의 결합도를 낮추게되면 그만큼 자연스럽게 높아지는게 있는데 그것은 바로

**테스트성** 이다.

객체에 대한 테스트를할때 실제 객체를 이용하는게 아니라 `test double` 객체로 교체해서 할 수 있다.

## 의존성 주입 방법

1. 생성자 주입
2. setter 주입
3. 인터페이스 주입(의존성 분리)

의존성 주입 방법에는 크게 3가지가 있다(framework도 있다는데 그건 필요성을 아직 모르겠다..)

일단 생성자 주입이랑 setter 메서드를 활용한 주입 방식은 위 코드에서 잠깐 살펴봤었는데 

마지막 방법인 인터페이스 주입은 뭘까??

## 의존성 분리

**의존성 주입**을 위해서 의존성을 분리하는 방법도 있다. 

의존성 분리란, `객체`와 `객체` 사이에 `인터페이스`를 끼워주면서, 한층 더 다이나믹하게 의존관계를 형성하는 것이라고 생각해도 될것 같다.

이걸 하기위한 방법으로는 객체지향프로그래밍의 5대원칙 중 하나인 **의존 역전 원칙** (Dependency Inversion Principle) 이 있다.

## 의존 역전 원칙(Dependency Inversion Principle)

**의존 역전 원칙**에 의하면 

1. 상위 계층은 하위 계층에 의존해서는 안된다. 두 계층 모두 추상화에 의존해야된다.
2. 추상화는 세부 사항에 의존해서는 안된다. 세부사항이 추상화에 의존해야된다.

다시 말해, 상위계층이 하위계층에 의존하게 되는 상황을 반전 시키기 위해서 하위 계층의 구현으로부터 독립되게 하는 것이다.

이걸 하기 위해서 **Interface** (스위프트에서는 `protocol` )를 사용한다.

```swift
// 의존 관계를 분리해주는 인터페이스
protocol Memory {}
protocol Processor {}
protocol OperatingSystem {}

// 위의 인터페이스에 의존하는 객체들
struct RAM: Memory {}
struct M1: Processor {}
struct MacOS: OperatingSystem {}

// 위 객체들과 의존관계가 있음
class 조립식PC {
	// 의존을 더이상 하위 클래스한테 안하고, 인터페이스와 하고 있다.
	private let ram: Memory
	private let processor: Processor
	private let os: OperatingSystem
	
	init(ram: Memory, processor: Processor, os: OperatingSystem) {
		self.ram = ram
		self.processor = processor
		self.os = os
	}
}

```

위 와 같이 프로토콜을 이용해서 객체간의 의존관계를 독립시키는 것이다.

이해를 하기위해서 만들어놓은 예시라서 실제 컴퓨터 구조와는 거리가 있을 수도 있겠지만, `M1` 클래스와 `Processor` 프로토콜 간의 관계를 한 번 살펴보자.

만약에 `M1` 클래스와 `Processor` 프로토콜이 이런식으로 되어있다고 가정하자

```swift
protocol Processor {
	let controlUnit: ControlUnit
	let alu: ALU
	let transistor: Transistor
}

struct M1: Processor {
	let controlUnit: ControlUnit
	let alu: ALU
	let transistor: Transistor
}
```

만약에 `M1` 객체가 `controlUnit` , `alu`, `transistor` 중에 하나를 빠뜨리면 오류가 날것이다.

왜냐하면 `Processor` 프로토콜을 채택하기로 했으면, 그 프로토콜에서 명시된 속성/기능들은 전부 구현을 해야되기 때문이다.

제어의 주체가 `M1` 이 아닌 `Processor` 라는 인터페이스에게 있기 때문에, 의존과 함께 제어 또한 반전이 된것을 볼 수 있다.

제어가 반전 되는 것을 (IOC-Inversion of Control) 이라고 하는데, 위 처럼 제어의 주체가 역전 되는 패턴이라고 볼 수 있다.

### 정리를 해보면

**인터페이스** 를 이용해서 상위 계층(`조립식PC`) 가 하위 계층(`ram`, `os`, `processor` ) 에 의존관계를 분리 시켰다.

위와 같이 인터페이스를 이용해서 의존성을 분리하는 것은 

**“Code to Interface not to implementation” 원칙** 와도 깊은 관련이 있고,

이또한 **의존성 주입** 이 주는 이점 중에 하나라고 볼 수 있다.

## 의존성 주입 정리

- 객체간의 결합도를 낮춘다.
- 객체간의 의존관계를 없애거나 변경할 수 있다.
- 두 객체간의 결합도를 낮춘다.
    - 유연성과 확장성 향상
    - 그럼 자연스럽게 테스트가 용이해진다
- 가독성이 높아진다.
- 코드 재사용성이 높아진다.
- “Code to Interface not to implementation” 원칙 지킬 수 있음

## 의문이 남은 부분

의존성 분리까지 해야 의존성 주입일까??

# 참고 자료

[https://stackoverflow.com/questions/130794/what-is-dependency-injection](https://stackoverflow.com/questions/130794/what-is-dependency-injection)

[https://en.wikipedia.org/wiki/Dependency_injection](https://en.wikipedia.org/wiki/Dependency_injection)

[https://mangkyu.tistory.com/150](https://mangkyu.tistory.com/150)

[https://medium.com/@jang.wangsu/di-dependency-injection-이란-1b12fdefec4f](https://medium.com/@jang.wangsu/di-dependency-injection-%EC%9D%B4%EB%9E%80-1b12fdefec4f)

[https://pizzasheepsdev.tistory.com/11](https://pizzasheepsdev.tistory.com/11)

---
title: "Error Handeling"

categories:
  - swift
tags:
  - [iOS, swift]

toc: true
toc_sticky: true

date: 2021-10-20
last_modified_at: 2021-12-3
---
# 에러 처리

## 에러 propagation

- 에러를 **propagate** 할때 **catch** 될때까지 다른 함수들을 통해 계속해서 전달 할 수 있다
    - `function 1` 에서 발생한 에러를 → `function 2` → `function3` ..이런식으로 계속 전달 할 수 있다는 뜻이다.
    - 코드로 공부를 한 번 해봤는데
    
```swift
    enum TestError: Error {
        case oddError(num: Int)
        case tooSmallError(num: Int)
    }
    
    func isEven(num: Int) throws -> Bool {
        guard num % 2 == 0 else {
            throw TestError.oddError(num: num)
        }
        return true
    }
    
    func isGreaterThan10andEven(num: Int) throws {
        guard try isEven(num: num) else {
            throw TestError.oddError(num: num)
        }
        guard num > 10 else {
            throw TestError.tooSmallError(num: num)
        }
    }
    
    func test(num: Int) -> Bool {
        do {
            try isGreaterThan10andEven(num: num)
        } catch TestError.oddError(num: let oddNum) {
            print("Input number \(oddNum) is odd")
            return false
        } catch TestError.tooSmallError(num: let smallNum) {
            print("Input number \(smallNum) is not bigger than 10")
            return false
        } catch {
            print("예상하지 못한 오류 \(error) 발생")
        }
        return true
    }
```
- 숫자가 `10` 보다 **크고** **짝수** 인지 확인하기 위해서 위와 같이 오류처리를 해봤다
- 일부러 두번째 함수 `func isGreaterThan10andEven(num: Int) throws {` 안에서
- `func isEven(num: Int) throws {` 를 `try` 해봤다
- 그러면 만약에 `isEven` 에서 오류가 trigger 되면, 그게 `isGreaterThan10andEven()` 으로 전달이 되고, 그대로 `func test(num: Int) -> Bool {` 함수까지 타고 올라가서 `catch` 되는 것을 볼 수 있다.
- 이때, 만약에 오류가 발생한 지점에서 숫자도 함께 전달하고 싶으면, `Enum` 에서 만들어줄때 value를 함께 만들어주면된다. 그럼 오류를 `throw` 할때 숫자도 함께 propagate 된다.
    - 이때, 오류를 `catch` 할때 `catch TestError.oddError(num: let oddNum) {` 이런식으로 숫자를 상수로 잡아내면 된다.
## init에서 사용하는 에러처리

- `init` 안에서도 오류 처리가 가능하다
- 이걸 확인해보기 위해 아래처럼 코드를 만들어봤다
        

```swift
func isEven(number: Int) throws -> Int {
    guard number % 2 == 0 else {
        throw TestError.oddError(num: number)
    }
    return number
}

struct EvenNumber {
    let num: Int
    init(num: Int) throws {
        self.num = try isEven(number: num)
    }
}

var numberInstance: EvenNumber?
do {
    numberInstance = try EvenNumber(num: 10)
} catch TestError.oddError(num: let num) {
    print("\(num)은 홀수 입니다")
}
if let evenNumber = numberInstance {
    print(evenNumber.num)
}
```

- `EvenNumber` 이라는 구조체의 생성자에서, 숫자가 짝수인지 확인을 해보고 생성을 할 수 있다.
- 그래서 만약에 `do-try-catch` 를 통해 `EvenNumber`를 시도 해보면, 짝수를 제대로 생성자에 넣으면 오류가 없고, 홀수를 넣으면 `isEven()` 에서 오류가 `throw` 되는 것을 볼 수 있다.
- 11을 생성자에 넣으면 "11은 홀수 입니다" 가 출력 되는 것을 볼 수 있고, 인스턴스는 생성되지 않는다.

---
title: "Keyboard Return 버튼"

categories:
  - uikit
tags:
  - [iOS, uikit]

toc: true
toc_sticky: true

date: 2022-1-27
last_modified_at: 2022-1-27
---

# Keyboard Return

키보드에서 이것저것 입력하고나서 리턴 버튼을 눌렀을때 다음 이벤트를 발생시키는 방법을 알아보자

`UITextFieldDelegate` 에 `textFieldShouldReturn` 메서드를 구현해주면 된다.

그럼 `resignFirstResponder()` 로 키보드 사라지게 할 수도 있고, 다음 `Responder` 를 찾게 만들 수도 있다.
[Keyboard 완료 버튼 구현](https://junbangg.github.io/uikit/Keyboard-Done-Button/) 에서도 다뤘는데 똑같은 로직 이용하면 된다.



```swift
extension ViewController: UITextFieldDelegate {
    func textFieldShouldReturn(_ textField: UITextField) -> Bool {
        let nextTag = textField.tag + 1

        if let nextResponder = textField.superview?.viewWithTag(nextTag) {
            nextResponder.becomeFirstResponder()
        } else {
            textField.resignFirstResponder()
        }
        return true
    }
}
```

이렇게하고 각 `UITextField` 의 `delegate` 를 `viewDidLoad` 에서 설정해주면 된다.

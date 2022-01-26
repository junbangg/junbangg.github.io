---
title: "Keyboard에 완료 버튼 추가"

categories:
  - uikit
tags:
  - [iOS, uikit]

toc: true
toc_sticky: true

date: 2022-1-27
last_modified_at: 2022-1-27
---

# Keyboard 에 완료 버튼을 구현하는 방법

![Screen Shot 2022-01-27 at 12 36 07 AM](https://user-images.githubusercontent.com/33091784/151197361-e574839e-9299-48c8-80c4-3542b0411cc7.png)


`UIToolBar` 에 버튼 하나 달아주면 되는데, 가장 오른쪽에 넣어주려면 빈공간을 왼쪽에 삽입해줘야된다.

이것도 버튼으로 구현하긴했는데 더 좋은 방법이 분명히 있을것 같아서 한 번 더 고민해봐야겠다.

## UI 코드

```swift
extension UITextView {
    func addDoneButton() {
        let screenWidth = UIScreen.main.bounds.width
        let doneToolBar = UIToolbar(frame: .init(x: 0, y: 0, width: screenWidth, height: 50))
        let emptySpaceItem = UIBarButtonItem(
            barButtonSystemItem: .flexibleSpace,
            target: nil,
            action: nil
        )
        let doneBarButtonItem = UIBarButtonItem(
            title: "완료",
            style: .done,
            target: self,
            action: #selector(dismissKeyboard)
        )
        let items = [emptySpaceItem, doneBarButtonItem]
        
        doneToolBar.items = items
        doneToolBar.sizeToFit()
        inputAccessoryView = doneToolBar
    }
}
```

## 완료 버튼을 누르면?

여기서 `#selector` 메서드를 어떻게 구현하느냐에 따라 할 수 있는게 많아진다.

단순히 keyboard만 없애고싶으면 다음과 같이 하면 될텐데.

```swift
@objc
private func dismissKeyboard() {
	resignFirstResponder()
}
```

여러개의 `UITextField` 에 입력을 하면서 완료버튼을 누르면 다음 `Responder` 를 찾는 로직도 구현할 수 있을것이다.

```swift
@objc
private func dismissKeyboard() {
    let nextTag = self.tag + 1
    
    if let nextResponder = self.superview?.viewWithTag(nextTag) {
        nextResponder.becomeFirstResponder()
    } else {
        resignFirstResponder()
    }
}
```

일단 이런 로직이 있으려면 모든 `UITextField` 의 `tag` 를 설정해줘야한다.

그러면 `nextTag` 를 이용해서 `UITextField` 를 찾아서 `becomFirstResponder()` 를 해주면 된다.

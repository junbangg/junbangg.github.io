---
title: "화면 터치하면 키보드 사라지게 하는 방법"

categories:
  - uikit
tags:
  - [iOS, uikit]

toc: true
toc_sticky: true

date: 2022-1-27
last_modified_at: 2022-1-27
---

# 화면 터치하면 키보드 사라지게 하는 방법
키보드 입력하다가 화면 어딘가를 눌렀을때 키보드 사라지게하는 마술은 다음과 같이 하면 된다.

```swift
extension UIViewController {
    func hideKeyboardOnTap() {
        let tap = UITapGestureRecognizer(target: self, action: #selector(dismissKeyboard))
        view.addGestureRecognizer(tap)
    }
    
    @objc func dismissKeyboard() {
        view.endEditing(true)
    }
}
```

이렇게 extension을 만들어놓고 필요한 `UIViewController` 의 `viewDidLoad` 에 넣어주면 된다.

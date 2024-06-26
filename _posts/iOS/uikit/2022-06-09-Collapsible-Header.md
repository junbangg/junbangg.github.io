---
title: "Collapsible Header 구현기"

categories:
  - uikit
tags:
  - [iOS, uikit]

toc: true
toc_sticky: true

date: 2022-06-09
last_modified_at: 2022-06-09
---

# 구현한 기능

![yogiyo-1](https://user-images.githubusercontent.com/33091784/172900969-a36297a5-1ca3-4d40-af87-ad3bd9ff21e7.gif)

위와 같은 스크롤 기능을 구현 하기위한 방법을 고민해봤습니다.

위로, 또는 아래로 일정한 만큼 이상을 스크롤하면, 가장 위에있는 헤더 뷰가 애니메이션과 함께 자동으로 이동하는 것을 볼 수 있습니다.

위 기능을 구현하기 위해서는 다음 두 가지 스텝이 필요하다고 생각하고 개발에 임했습니다.

1. 스크롤을 감지하는 기능
2. 스크롤이 감지됐을때 애니메이션이 트리거 되는 기능

하나씩 구현한 방법을 소개하겠습니다.

## 스크롤을 감지하는 기능

스크롤이 얼만큼 됐는지 감지하는 기능이 필요한데, `UIScrollViewDelegate` 에서 제공하는 `scrollViewDidScroll` 이라는 메서드를 사용했습니다.

[애플 공식문서](https://developer.apple.com/documentation/uikit/uiscrollviewdelegate/1619392-scrollviewdidscroll)를 보면 

![Screen Shot 2022-07-04 at 1 30 08 AM](https://user-images.githubusercontent.com/33091784/177048714-5a282ba1-2772-4370-bfd9-7390bc3d0aba.png)


`Tells the delegate when the user scrolls the content view within the receiver` 

해당 기능을 구현하기위해 필요한 메서드라고 판단했습니다.

사용자가 스크롤을 얼마나 했는지 판단하기 위해 다음과 같이 구현했습니다.

![yogiyo-1](https://user-images.githubusercontent.com/33091784/172901031-13cc2542-6e54-475d-b2ea-6bb927f367c0.gif)
<img width="450" alt="Screen Shot 2022-06-07 at 5 33 43 PM" src="https://user-images.githubusercontent.com/33091784/172899444-a9bd11d6-004f-49bc-84f6-c3fd3847df87.png">


`scrollViewDidScroll` 를 이용해서 사용자가 얼만큼 스크롤을 했는지 알아내서, 그만큼 `header` 의 `constraint` 에 적용을 하는 방법으로 구현했습니다.

## contentOffset

스크롤이 어느 방향으로 얼만큼 됐는지는 `contentOffset` 이라는 `scrollView` 프로퍼티를 이용하면 됩니다.

![Screen Shot 2022-06-10 at 12 59 22 AM](https://user-images.githubusercontent.com/33091784/172900034-700f13c2-1e7b-45d4-9bed-23b8a245d405.png)

`contentOffset` 이란 `scrollView` 의 `contentView` `frame` 과의 상대적인 위치입니다.

그림으로 표현하면 다음과 같습니다.

<img width="574" alt="Screen Shot 2022-06-10 at 1 09 03 AM" src="https://user-images.githubusercontent.com/33091784/172900083-4199631b-96bd-48fd-9587-f4b5b21d0624.png">

[https://stackoverflow.com/a/39062209](https://stackoverflow.com/a/39062209) 참고

`contentOffset` 의 `y`  좌표 값을 이용해서 스크롤을 계산할 수 있습니다.

## 스크롤에 따른 UI 변경

`contentOffset` 의 스크롤이 위로 됐는지, 또는 아래로 됐는지, 그리고 얼만큼 스크롤이 됐는지를 판별했으면, 그에 따른 `UI` 변경을 수행하는 로직은 다음과 같습니다.

<img width="466" alt="Screen Shot 2022-06-10 at 12 26 30 AM" src="https://user-images.githubusercontent.com/33091784/172900334-a5421410-d091-41c4-b28d-078fbf077148.png">


`scrollView.contentOffset.y` 에 따라서 스크롤이 위로 갔는지 아래로 갔는지 판단할 수 있는데

- `30` 을 초과했으면 `shouldSnapUp`
    - 이때 `header` 뷰를 위로 움직이는 애니메이션을 실행시킵니다
- `0` 이하이면 밑으로`isSwipingDown`
    - 이때 `header` 의 `alpha` 값을 이용해서 뷰를 투명하게 변경시킵니다.
    - 변경하지않으면 다음과 같은 부자연스러운 현상이 일어납니다.

<img width="348" alt="Screen Shot 2022-06-10 at 1 27 57 AM" src="https://user-images.githubusercontent.com/33091784/172900390-d8dbb160-e030-4dfb-9c7d-357c4fd218c8.png">

### `shouldSnapUp` 일때는 다음 두가지 일이 일어나는데

1. `headerView` 를 위로 이동시킨다
    
    ```swift
    private func snapUpHeaderView(by height: CGFloat, when shouldSnap: Bool) {
        self.headerViewTopConstraint?.constant = shouldSnap ? -height : 0
    }
    ```
    
2. `navigationBar` 를 숨긴다
    
    ```swift
    private func hideNavigationBar(when shouldHide: Bool) {
        self.navigationController?.navigationBar.isHidden = shouldHide
    }
    ```
    

이 모든 `UI` 변경을 애니메이션과 함께 수행하는데

`UIViewPropertyAnimator.runningPropertyAnimator` 를 사용하여 한번에 해결했습니다.

![Screen Shot 2022-06-10 at 1 28 10 AM](https://user-images.githubusercontent.com/33091784/172900439-19f8d51d-62e9-4247-a968-93fb7375838d.png)

`UIViewPropertyAnimator` 는 뷰에 변경사항을 애니메이션과 함께 적용시킬 수 있고, 애니메이션에게 동적으로 변경사항을 줄 수 있는 클래스이며 

![Screen Shot 2022-06-10 at 1 45 59 AM](https://user-images.githubusercontent.com/33091784/172900605-20813877-eecd-4c41-a585-ecfd5a9d9cab.png)


`runningPropertyAnimator` 는 애니메이션을 바로 수행시키는 클래스를 리턴하는 메서드입니다.

최종적으로 다음과 같이 애니메이션을 실행시켰습니다.

```swift
UIViewPropertyAnimator.runningPropertyAnimator(
            withDuration: 0.3,
            delay: 0,
            options: [],
            animations: {
            self.snapUpHeaderView(by: headerViewHeight, when: shouldSnap)
            self.hideNavigationBar(when: !shouldSnap)
            self.view.layoutIfNeeded()
        })
```

### `isSwipingDown` 일때는 `headerView` 를 투명하게 만듭니다

```swift
private func applyAlpha(when isSwipingDown: Bool) {
    let transparencyDegree = 1.0
    
    UIView.animate(withDuration: 0.3) {
        self.headerView.alpha = isSwipingDown ? transparencyDegree : 0
    }
}
```

`UIView.animate` 를 사용해서 `header` 의 `alpha` 값을 변경시켰습니다.

---
title: "UICollectionView 에서 Image Cell 지우는 방법"

categories:
  - uikit
tags:
  - [iOS, uikit]

toc: true
toc_sticky: true

date: 2022-1-27
last_modified_at: 2022-1-27
---

# UICollectionView 에서 cell 지우는 방법

당근마켓, 번개장터 같은 앱에서 제품 사진을 등록할때 다음과 같은 `UICollectionView` 형태를 볼 수 있는데

![Screen Shot 2022-01-26 at 11 53 48 PM](https://user-images.githubusercontent.com/33091784/151191372-b52b4f25-e274-4abd-94c2-8bde5d4533e8.png)


이때 각 사진에 있는 x 버튼을 누르면 `collectionView` 에서 삭제가 되는 방법을 한 번 정리 해둬야겠다.

## Delegate 패턴
`UICollectionCell` 에서 버튼이 눌리면, Delegate 패턴을 이용해서 `ViewController` 가 `collectionView`에서 해당 `index` 의 `cell` 을 삭제하게 만들면 된다.

## Delegate 코드
```swift
protocol RemoveImageDelegate: AnyObject {
    func removeFromCollectionView(at index: Int)
}
```

## UICollectionViewCell 클래스

다른 방법이 있을지는 모르겠지만, `ImageCell` 이 `index` 에 대한 정보를 가지고 있게 구현해봤다.
그래서 삭제 메서드가 호출이 되면, `ViewController` 에서 `index` 를 찾아서 해당 `cell` 을 삭제하게 만들면 된다.
그리고 `collectionView`와 연동된 데이터 배열에서도 해당 데이터는 삭제를 해줘야하기 때문에 이 `index` 정보는 가지고 있어야된다고 생각했다.


```swift
 final class ImageCell: UICollectionViewCell {
    private var index: Int?
    weak var delegate: RemoveImageDelegate?
    
    ...

    private func setupDeleteButton() {
        self.deleteButton.addTarget(
            self,
            action: #selector(deleteImage),
            for: .touchUpInside
        )
    }
    
    @objc func deleteImage() {
        guard let index = index else {
            return
        }
        delegate?.removeFromCollectionView(at: index)
    }
}
```


`ImageCell` 은 `ViewController` 가 가지고있는 데이터 배열의 인덱스를 프로퍼티를 가지고 있고,

버튼이 눌리면 `delegate?.removeFromCollectionView(at: index)` 를 호출해서

다음과 같이 `collectionView` 에서 삭제해주면 된다.

## ViewController 코드

```swift
extension ViewController: RemoveImageDelegate {
    func removeFromCollectionView(at index: Int) {
        self.collectionView.deleteItems(at: [IndexPath.init(row: index + 1, section: 0)])
        self.images.remove(at: index)
    }
}
```
이때, `collectionView` 에서 삭제하는거랑 `images` 배열에서 삭제하는 순서가 중요하다.

images 를 먼저 지우면 `index` 오류가 발생할 수 있다.


![Screen Shot 2022-01-27 at 12 19 35 AM](https://user-images.githubusercontent.com/33091784/151191432-f6f1c45b-ddde-4cb9-9f5f-88acfc966894.png)

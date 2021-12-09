---
title: "TableView-Datasource"

categories:
  - uikit
tags:
  - [iOS, uikit]

toc: true
toc_sticky: true

date: 2021-12-7
last_modified_at: 2021-12-7
---


# TableView

`TableView`는 iOS 에서 데이터를 표현하기 위한 대표적인 방법이다.

`TableView` 를 이루는 구성요소에는 크게 4가지가 있다.

- `Cell` : 화면에 표시하려는 데이터를 담는 틀 (`row`)
- `TableViewController` : 테이블뷰를 관리하기위한 객체. 보통 `UITableViewController` 사용
- `Datasource` : 테이블에 표시할 데이터를 제공하는 객체
- `Delegate` : 테이블과 유저간의 interaction을 관리해주는 객체

그래서 요 녀석들을 이용해서 `TableView` 를 구현하는건데, 하나씩 살펴봐야겠다.

# UITableViewDataSource

![Screen Shot 2021-12-08 at 8 15 47 PM](https://user-images.githubusercontent.com/33091784/145199347-0b497c91-d54d-48d8-8d05-54cea481e0d9.png)


`TableView` 를 사용해서 표현하는 데이터들을 동적으로 만들어줘야하는 경우가 대다수인데,

이걸 `UITableViewDataSource` 프로토콜을 채택한 객체를 이용해서 한다.

`TableView` 를 그리기 위해서는 두가지의 데이터 반드시 필요한데, 바로

1. **테이블뷰에 표시할 `Section` 의 수와 `Row` 의 수**
2. **화면에 보이는 테이블뷰 부분에 표시할 `Cell`** 

이 두가지이다. 

그렇기 때문에 이 `Protocol` 을 채택하면 

```swift
class Datasource: NSObject, UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        
    }
}
```

이렇게 두개의 메서드는 필수적으로 구현해줘야한다

그럼 필수 정보를 어떤식으로 제공하는지 하나씩 살펴보자.

<br>


## 테이블뷰에 표시할 `Section` 의 수와 `Row` 의 수
`tableView` 필수 정보의 첫번째인 `Section` 의 수와 `Row` 의 수는 아래의 메서드를 활용해서 제공할 수 있다.
```swift
func numberOfSections(in tableView: UITableView) -> Int
func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int
```

공식문서에 의하면, 위 두 메서드에서 리턴을 최대한 **빠르게** 해줘야된다고 한다.

이유를 생각해보면, `TableView` 가 그려지려면 `Section` 수, `Row` 수가 필요하다고 했는데, 그걸 느린 연산으로 가져오면 뷰가 그려지는 속도가 늦어지니까 그런것 같다.

그래서 추천하는 방법이 **배열** 을 사용하는것이다. `TableView` 의 데이터구조랑 배열의 구조를 맞추면  `Section` 수, `Row` 수도 빠르게 접근할 수 있기 때문인것 같다.

<br>


## 화면에 보이는 테이블뷰 부분에 표시할 `Cell`

**`Cell`** 은 다음과 같은 방법으로 제공해줄 수 있다.

일단 `UITableViewDataSource` 에 있는 
```swift
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {} 
```


메서드를 사용해서 `Cell` 을 제공해주는건데. 아래와 같이 보통 구현하는것 같다.

```swift
func tableView(_ tableView: UITableView,
                 cellForRowAt indexPath: IndexPath) -> UITableViewCell {
  
   let cell = tableView.dequeueReusableCell(withIdentifier: "cell",
                                                                                     for: indexPath)
   cell.textLabel!.text = "Row \(indexPath.row)"
   return cell
}
```
여기서 중요한 특징은, **`tableView`가 한번에 모든 `cell` 을 요구하지않는다는 것이다.** `lazy` 하게 `cell` 을 달라고하는데, 화면에 보이는 부분에 대해서만 위의 `cellForRowAt` 메서드로 `cell`을 달라고하고, 그 밑에 아직 보이지않는 `cell` 들은 아래로 스크롤을 하면서 요구한다.

사용자가 아래로 스크롤을하면, 위쪽에 있는 `cell`들은 위쪽으로 사라지면서 `tableView`가 가지고있는 `reuseQueue` 에 들어가게 된다.
그리고 밑에서 새롭게 올라오는 `cell`들은 `dequeueReusableCell` 메서드를 통해서 `reuseQueue` 에서 꺼내와서 재사용하여 새로운 데이터를 담는 `cell` 이 되는 것이다. 

![Table View-1](https://user-images.githubusercontent.com/33091784/145315887-b2974d63-4596-4bf6-bdb0-d17d4b7b2cf9.jpg)

이렇게되면 매번 `UITableViewCell` 인스턴스를 새롭게 생성할 필요 없이 기존에 생성했던 인스턴스들을 **recycle** 할 수 있는것이다.

엄청나게 많은 데이터를 앱으로 가져와서 `tableView` 로 표시해야되는 상황을 상상해보면 왜 `lazy`하게 하는건지 좀 짐작이 가는것 같다.
유저가 아래로 스크롤을 할지 안할지도 모르는데 미리 그 많은 데이터를 `cell`로 만들어놔서 담아놓는것은 굉장히 비효율적일것이다.

다만, 위의 동작 방법대로라면, 화면에 표시되는 범위의 `UITableViewCell` 의 개수만으로 계속 재사용되는건지 궁금해진다.
**예를 들어 화면에 보이는 `cell` 이 20개 정도라고 하고, 해당 화면에 보여질 수 있는 데이터의 개수가 1000개 정도 된다고 했을때, 20개의 `UITableViewCell` 로만 1000개가 표현되는건지? 궁금했는데,**
**그건 상황마다 다르다고 하는것 같고 정확히 알 수 없는것 같다. 스크롤을 천천히 내리면, 재사용큐에 들어갔던게 바로 나와서 새로운 `UITableViewCell` 이 안나올 수도 있는데, 빠르게 스크롤하면 그럴 시간이 없을것 같다.**

그리고 위에 그림을 보면, `dequeueReusableCell` 로 꺼내져온 `cell` 이 `datasource` 한테 전달되기 전에, 잠깐 거치는 곳이 있는데,
바로 `prepareForReuse` 라는 메서드이다.


### `prepareForReuse`

![Screen Shot 2021-12-08 at 8 16 08 PM](https://user-images.githubusercontent.com/33091784/145199401-5a0a86ed-39b4-4f3c-9471-fd7005a2ef51.png)

`prepareForReuse` 가 하는 일은 말 그대로다.
메서드로 들어오는 `UITableViewCell`을 다시 **configure** 할 수 있게 **reset** 해주는 것이다.
공식문서에 의하면, 성능적인 측면 때문에 여기서는 **content** 과는 무관한 것들만 **reset** 해주는 것이라고 한다.

이 부분에 대해서 생각해보면서 `cell` 에 이미지가 있고 재사용할 때마다 이미지를 바꿔줘야하는 상황은 어떨지 상상을 해봤는데
<https://stackoverflow.com/questions/50916739/reusing-cells-in-uitableview>
`prepareForReuse` 를 `override` 하여 `default` 이미지를 세팅 해주면 성능 저하가 해결 될 수 있다는 이야기가 있는것 같다.
이 부분은 프로젝트에서 한 번 실험 해봐야될것 같다.

### `dequeueReusableCell()`

여기서 또 다른 핵심은 `dequeueReusableCell(withIdentifier identifier: String)`이라는 메서드인데. 이 메서드가 하는 일은
    
- 일단 `tableView` 가 갖고있는 큐에 접근을해서 들어있는 `UITableViewCell` 이 있는지 확인을 한다.
    
    1. 있으면 그걸 리턴한다
    2. 없으면, `withIdentifier` 로 들어온 `identifier` 를 이용해서 새로운 "빈" `UITableViewCell` 을 리턴한다.
    
    - 이 메서드도 매개변수만 살짝 다른게 있는데
        - `func dequeueReusableCell(withIdentifier identifier: String -> UITableViewCell?`
            - 아까 위에거는 `dataSource` 에서만 사용을 해야되는 반면에, 이건 밖에서도 사용이 가능하다는 차이점이 있다.
            - 그리고 이 메서드는, `reuse queue` 에 `cell` 도 없고, 유효한 `identifier` 도 없으면 `nil` 을 리턴한다.
        
       
        저거 두개중에서 `identifier` 이랑 `indexPath` 둘다 있는 메서드가 새롭게 생긴 메서드라고 한다.
        **근데 이 두개의 메서드는 어떨때 쓰임이 다른지 아직 잘 와닿지는 않는다.**
        
        
- 그러면 밖으로 나온 **Reusable Cell** 을 configure해서 테이블뷰 한테 최종적으로 주는거다.

## 궁금한점

성능을 위한 Prefetching이 어떻게 구현되는지 궁금하다

[Apple Developer Documentation](https://developer.apple.com/documentation/uikit/uitableviewdatasourceprefetching)

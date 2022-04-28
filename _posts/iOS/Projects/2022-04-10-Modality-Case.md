---
title: "Modality에 대한 고민🤔"

categories:
  - Projects
tags:
  - [iOS, Projects]

toc: true
toc_sticky: true

date: 2022-04-10
last_modified_at: 2022-04-10
---


# Modality의 적절한 사용

저희 얼리버디 에서는 사용자가 일정등록 할때 경로를 추가하는 기능을 제공하고있습니다. 

하지만 처음 어플리케이션을 기획했을때, 경로 검색 기능에 대한 UI 디자인에 `Modality` 를 과도하게 적용했다는 사실을 발견했습니다.

다음은 `Modality` 가 부적절하게 사용된 얼리버디의 모습입니다.

![Apr-25-2022 23-50-13](https://user-images.githubusercontent.com/33091784/165751608-0e241851-fb44-4ca9-aa31-39f1f67c05ae.gif)



`일정 등록` 페이지에서 경로 선택을 완료할때까지의 페이지 이동이 총 3번 발생하는데, 모든 이동이 `Modal` 을 통해 이루어진다는것을 볼 수 있습니다.

애플의 [HIG(Human Interface Guidelines)](https://developer.apple.com/design/human-interface-guidelines/ios/app-architecture/modality/) 공식 문서를 참고하면서 위와 같은 App Architecture 는 잘못되었다는 결론을 내렸습니다.

![Screen Shot 2022-04-28 at 9 27 12 PM](https://user-images.githubusercontent.com/33091784/165751725-f88ad027-b43d-4c12-a55c-ff62dc26cdfc.png)



위 문서 내용을 보면, `Modality` 는 어플리케이션 컨텐츠를 임시적으로 사용자에게 보여줄 때를 위한 방법이라고 명시 되어있습니다.

`Modality` 가 일반적인 `Navigation` 과 차별점이 있는 것은 바로 `temporary` 라는 개념이라고 생각합니다.

일반적인 `Navigation` 은 컨텐츠의 `depth` 에 따른 계층구조가 있는 반면, `Modality` 는 기존의 계층 구조에서 `임시적으로` 벗어날때 사용되는 것으로 구분짓는것이 맞다고 생각했습니다.

즉, 사용자의 주목이 임시적으로 필요한 경우에만 사용하는것이 적절하다고 생각했습니다.

위 결론에 따른 `Navigation` 변경 사항을 팀과 함께 논의한 뒤에 다음과 같이 개선했습니다.

![Apr-26-2022 01-01-46](https://user-images.githubusercontent.com/33091784/165751416-7794a6b9-5d47-4095-b19f-0f32dd5f4434.gif)

사용자가 경로 선택을 확인하는 곳에서만 Modal 을 사용하는 형태로 바꾸면서 애플의 HIG에 더 맞는 앱 디자인이 되었다고 생각합니다.

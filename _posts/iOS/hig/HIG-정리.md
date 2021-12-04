---
title: "HIG 간단 정리"

categories:
  - hig
tags:
  - [iOS, hig]

toc: true
toc_sticky: true

date: 2021-12-5
last_modified_at: 2021-12-5
---
## HIG

1. **Modality** 와 **Navigation**의 차이점
    
    ### Modality
    
    [Modality - App Architecture - iOS - Human Interface Guidelines - Apple Developer](https://developer.apple.com/design/human-interface-guidelines/ios/app-architecture/modality/)
    
    ### Navigation
    
    [Navigation - App Architecture - iOS - Human Interface Guidelines - Apple Developer](https://developer.apple.com/design/human-interface-guidelines/ios/app-architecture/navigation/)
    
    - 내 생각에는 원래의 **Navigation** 방식과는 별개로, 임시로 나타내야하는 뷰가 있으면 그게 **Modality 이다.**
    - 야곰: 정보의 흐름의 기준.
        - **Navigation**은 depth에 따라 계층구조가 있음
        - **Modality**는 원래의 계층구조에서 *잠깐* 벗어나야할때
    - 회원가입 페이지는 **Navigation**으로 하는게 맞을까 **Modality** 로 하는게 맞을까?
        - **Modality**가 맞다고 생각. 왜냐하면, 회원가입이라는 임시적인?기능을 사용자한테 시키고, 기존 앱의 흐름으로 돌아가는게 맞다고 생각함.
2. **Hierarchical** vs **Flat** Navigation
    - **Hierarchical 는 부모-자식 관계가 있음**
        - ex: 환경설정
    - **Flat**는 depth 없이 independent 한 잠재적인 부모 뷰들?
        - 음악앱

 3. **Text Field**와 **Text View**의 차이점

### Text Field

- 한줄 짜리 텍스트
- Text View와는 다르게 다음줄로 넘어가질 않음
    - ex: 이메일, 전화번호 입력

### Text View

- 여러 줄을 작성하고 편집할때 사용.
- Text Field와는 다르게 자동으로 다음줄로 넘겨줌
    - ex: 메모
1. **Popup Button**과 **Pulldown** Button의 차이점
    - **Popup Button 은 mutually exclusive options 로 contents가 update가 됨**
        - 예: 음악 앱에서 Sort Button
        
        ![IMG_0736.jpg](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d85796b1-9c96-47f5-8841-6761ced7eb70/IMG_0736.jpg)
        
    - **Pulldown** Button은 Button의 목적과 연관된 메뉴들을 표시
        - 사람이 어떤 선택을하던간에 contents의 변화는 없음
        - ex: 메모앱에서 카메라 pulldown
        
        ![IMG_0737.jpg](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d84b3fa5-4954-411b-beaa-54bec7da5a1d/IMG_0737.jpg)
        
    
    [Buttons - Controls - iOS - Human Interface Guidelines - Apple Developer](https://developer.apple.com/design/human-interface-guidelines/ios/controls/buttons/)
    
   
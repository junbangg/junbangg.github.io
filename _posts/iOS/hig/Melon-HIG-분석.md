---
title: "Melon HIG 분석"

categories:
  - hig
tags:
  - [iOS, hig]

toc: true
toc_sticky: true

date: 2021-12-5
last_modified_at: 2021-12-5
---
## Navigation

- Hierarchical Navigation

## Home 화면

- 화면 우측 상단에 Navigation Bar (이용권,           Melon 제목,      Search, My, Menu)
    - Search
        - 자식 뷰로 이동
        - 상단에
            - Search Bar(검색어를 입력하세요.)
            - My 폴더 버튼
            - Menu 버튼
        - 최근 검색어 Table View
    - My
        - 자식 뷰로 이동(Navigation Bar에 뒤로가기가 생긴다)
        - 좋아요, 최근들은, 나만의 차트, 플레이리스트, 팬맺은 아티스트 로 나뉜 Table View들이 나타난다
    - Menu
        - Popover 뷰
            - 상단에 (내 계정, 환경 설정 톱니바퀴 버튼)
            - 멜론 홈
            - 최신 음악
            - 멜론 차트
            - 장르 음악
            - For U
            - 멜론 DJ
            - 멜론 TV
- 여러개의 Collection View로 구성됨
    - 오늘의 추천선곡
        - 좌우로 스크롤할 수 있는 Scroll View
    - 최신음악
        - Foldable
    - Top 100
    - For U
    - 멜론 TV
    - 장르 음악

- 하단에 재생 bar?
    - 왼쪽에 재생목록 버튼
        - 누르면 Split View 로 재생 목록이 나온다.
    - 이 bar는 앱 어느 화면으로 이동하던 나타난다
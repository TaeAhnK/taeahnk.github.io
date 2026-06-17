---
layout: post
title: "softeon-theme"
display_area: other-projects
display_order: 25
thumbnail_type: img
thumbnail_img: "assets/images/projects/softeon-theme/softeon-theme.png"
thumbnail_vid: ""
tags:
  - Jekyll
  - Blog Theme
  - Vibe Coded
---

<!-- preview -->

<div align="center"><img src="/assets/images/projects/softeon-theme/softeon-theme.png" width="100%" height="auto"></div>

## About
- 2026.06
- Jekyll
- 1인 프로젝트
- Vibe Coded (Codex)
- 포트폴리오에 사용하기 위한 Jekyll Theme
- [Github](https://github.com/TaeAhnK/softeon-theme)
- [개발일지](https://code-in-coffee.tistory.com/55)
- [Sample Page](https://taeahnk.github.io/softeon-theme)

## Functions
- 반응형 포트폴리오 인덱스 페이지
- 라이트 / 다크 컬러 모드
- Jekyll Collection 기반 프로젝트 포스트 관리
- 프로젝트 표시 개수 및 그리드 컬럼 수 설정 기능
- Projects, Other Projects, Current Projects 섹션 분리
- 인덱스 페이지와 프로젝트 게시글을 결합한 인쇄용 뷰 구현
- A4 PDF 출력을 위한 인쇄 전용 CSS
- 화면에서는 영상 썸네일을 표시하고, 인쇄 화면에서는 대체 이미지를 표시하도록 구현

## Epilogue

웹에서의 바이브 코딩은 게임보다 훨씬 많이 발전한 것 같아 포트폴리오에 사용할 Jekyll Blog Theme을 새로 만들어 보았습니다. 기존에는 다른 사람이 올려둔 Blog Theme을 수정해 사용하고 있었고, 괜찮았지만 아쉬움이 있었습니다. 나아가 pdf 제출을 원하는 회사에 지원하는 경우, 포트폴리오 글을 복사해 새로 입력하는 번거로움을 줄일 수 있도록 웹사이트를 적절한 형태의 pdf로 출력할 수 있는 기능을 구현하고 싶었습니다.

먼저 필요한 기능을 정리하고, 구조를 구현한 뒤, Stitch에서 디자인했습니다. 디자인 파일을 만들어둔 구조에 적용하고, 세부 조정해 완성했습니다. html, css, Jekyll에 대해 기본적인 것은 알고 있지만, 자세히는 알지 못해 대부분 Codex를 사용해 구현했습니다.

완성 후 느낀 점은, 먼저 매우 놀랐습니다. 분명 웹에 대한 이해가 부족한 상태임에도 Codex에 원하는 기능을 요구하면 그럴듯한 기능을 만들어 주었습니다. 물론 한계가 있다고는 생각합니다. 정적 웹사이트는 잘 작동하고 예쁘게 보이기만 하면 되기 때문에 AI가 구현하기 쉬운 편이지만, 서버가 있는 웹사이트나 게임에서는 이렇게까지 쉽게 만들지는 못할 것 같습니다.

기술에 대한 이해 없이 썼을 때의 한계도 드러났습니다. css를 잘 모르니 간단한 css 수정과 롤백도 다 Codex에 맡기게 되었습니다. 수정을 위해 필요한 파일의 위치를 모르니 프로젝트를 다 검색하는 경우도 자주 있었습니다. 로직을 설명하며 구현하라고 하면 더 정확하고 저렴하게 구현하지만, 내가 원하는 기능을 묘사만 하고 만들어 달라고 하면 확실히 토큰 소모량이 많았습니다.

이런 상황에서 개발자는 기술을 이해하는 상태에서 원하는 내용을 로직으로 풀 수 있어야 한다고 생각합니다. "A 기능을 구현해 줘."를 넘어 "A 기능을 구현하려고 하는데, 이러한 조건이 있고, B 로직으로 작동하도록 만들어줘."라는 명령을 할 수 있어야 예측할 수 있고 확장할 수 있는 프로그램을 만들 수 있을 것입니다.
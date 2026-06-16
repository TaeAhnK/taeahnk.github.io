---
layout: post
title: "RoomGenerator"
display_area: other-projects
display_order: 20
thumbnail_type: img
thumbnail_img: "assets/images/projects/RoomGenerator/RoomGenerator.png"
thumbnail_vid: ""
tags:
  - Unreal 5
  - Plugin
  - 맵_자동생성
---

<!-- preview -->
## Demo

<div class="youtube-embed">
  <iframe 
    src="https://www.youtube.com/embed/--yLTI75p5o?si=9ZKbl74EDjVA0Ytg" 
    frameborder="0" 
    allowfullscreen="true">
  </iframe>
</div>

<h2> About </h2>
- 2026.05
- Unreal Engine, C++, SlateUI 사용
- 1인 프로젝트
- Vibe Coded
- 진행 중인 프로젝트에 사용할 Room 제작을 위해 만든 랜덤 RoomGenerator
- [Github](https://github.com/TaeAhnK/RoomGenerator)

## Fuctions
- Modular 에셋을 사용해 자동으로 바닥과 벽, 문, Decor를 설치
- Offset, 등장 빈도 등 설정 가능
- 시드 기반으로 시드를 변경할 때마다 다른 결과 생성
- 생성 결과를 `InstancedStaticMeshComponent`로 저장

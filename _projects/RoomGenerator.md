---
layout: post
title: "RoomGenerator"
thumbnail: "assets/img/RoomGenerator/RoomGenerator.png"
main_post: false
order : 2
---

#UnrealEngine #Plugin #맵_자동생성

<!--more-->
## Demo

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; margin: 0 auto;">
  <iframe 
    style="position: absolute; top: 0; left: 50%; width: 90%; height: 90%; transform: translateX(-50%);" 
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

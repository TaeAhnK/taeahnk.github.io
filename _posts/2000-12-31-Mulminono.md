---
layout: post
title: "Mulminono"
date: 2000-12-31 11:00:00
thumbnail: "assets/thumbnails/Mulminono.png"
preview_file: "_posts/previews/2000-12-31-Mulminono-preview.md"
main_post: false
---
<h2> About </h2>
- 2024.10.10
- Python
- ChatGPT를 사용한 Vibe Coding
- 멀미 방지를 위한 사각형 인디케이터를 표시해주는 프로그램
- [Github](https://github.com/TaeAhnK/MulmiNoNo)

## 구현 목표
- 아이폰의 '차량 모션 큐' 기능처럼, 스크린에 점이나 포스트잇을 붙여 3D 멀미를 예방할 수 있다는 정보가 있습니다.
- 번거롭게 포스트잇을 붙이는 대신, 프로그램으로 화면에 사각형을 그려 포스트잇의 역할을 하도록 하였습니다.
- <details><summary></summary><div align="left"><img src="/assets/img/Mulminono/Mulminono01.png" width="40%" height="auto"></div></details>


## 구현 결과

<div align="left"><img src="/assets/img/Mulminono/Mulminono02.png" width="30%" height="auto"></div>

Python의 tkinter를 사용한 프로토타입 코드를 바탕으로, 디테일을 추가해 사각형을 그리는 기능을 완성했습니다.   
이후, 개인적으로 사용하며 필요한 기능을 추가하였습니다.

| 단축키 	| 기능                   |
|----------|---------------------- |
| Q        	| 종료                  |
| B        	| 검은 사각형          	|
| W        	| 흰 사각형         |
| +/-      	| 사각형 크기 조절  |
| 1        	| 꼭짓점에 사각형   |
| 2        	| 변에 사각형       |
| 3        	| 사각형 8개        |
| Space    	| 보이기/숨기기     |

<br>

<div align="center"><img src="/assets/img/Mulminono/Mulminono03.png" width="60%" height="auto"></div>


## 개발 일지
[[Mulminono] Chat GPT와 함께 3D 멀미 완화 프로그램 만들기](https://code-in-coffee.tistory.com/45)


## Epilouge
AI의 코딩 실력이 빠르게 향상되면서, 이제는 단순한 프로그램이나 프로토타입을 AI를 통해 손쉽게 만들어낼 수 있게 되었습니다.
AI가 생성한 코드를 바탕으로 세부 사항을 직접 수정하고 기능을 확장함으로써, 개발에 소요되는 시간을 크게 줄일 수 있다고 생각합니다. 빠르게 발전하는 LLM을 적극적으로 활용해 개발 효율을 높이는 한편, AI에 지나치게 의존하지 않고 자유롭게 변형하고 응용할 수 있는 개발자가 되고자 합니다.
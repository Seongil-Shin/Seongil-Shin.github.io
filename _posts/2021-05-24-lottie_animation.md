---
title: 로티(lottie) 애니메이션 적용
author: 신성일
date: 2021-05-24 21:38:44 +0900
categories: [study, css]
tags: [CSS, front end, lottie]
---

## lottie 애니메이션

Junction 해커톤을 하며 lottie 애니메이션을 접하게 되었다.

간단히 코드로 불러올 수 있으며, json 파일로 불러올 경우 색상 변경 등의 커스터마이징도 가능하여 활용도가 높다.

## 불러오는 법(react 기준)

1. npm install @lottiefiles/react-lottie-player

2. 다음 코드 삽입

   ```js
   <Player
     src={fireVideo}
     background="transparent"
     speed="1"
     style={{
       position: "absolute",
       top: "230px",
       left: "257px",
       width: "150px",
       height: "150px",
       color: palette.red,
       opacity: 0.2,
       filter: "blur(1px)",
     }}
     renderer="svg"
     loop
     controls
     autoplay
   ></Player>
   ```

   이때 src 에는 로컬에 있는 json 파일 또는 링크가 들얼 수 있다.

## 색상 커스터마이징 방법

1. https://lottiefiles.com/tools/json-editor 에 접속하여 원하는 로티 애니메이션의 json 파일을 붙여넣는다.

2. Toggle source/editor 를 클릭한다.

3. ADBE Vector Graphic - Stroke 를 검색하고, 같은 객체에 들어있는 k 배열을 찾는다.
   ![image-20210524174313618](/assets/img/lottie costumise.PNG)
4. 0~2번째 원소는 각각 RGB 값을 담당한다. 계산방법은 다음과 같다

   #123456 코드가 있으면, 이것을 앞에서부터 2개씩 끊고 다음과 같이 계산

   12 -> 1 \* 16 + 2 -> 18 / 255 = 0.07058823529

   34 -> 3 \* 16 + 4 -> 52 / 255 = 0.20392156862

   56 -> 5 \* 16 + 6 -> 86 / 255 = 0.33725490196

5. 위에서 계산한 코드를 앞에서부터 차례대로 넣으면 색상 커스터마이징 완료

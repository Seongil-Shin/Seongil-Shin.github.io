---
title: 브라우저 렌더링 과정
author: 신성일
date: 2023-05-30  22:00:00 +0900
categories: [study, web]
tags: [browser]
---



## 렌더링 순서

웹 페이지에 접속하면 브라우저는 제일 먼저 해당 페이지의 `html`을 불러온다. 이후 가져온 html을 읽으며 다음과 같은 렌더링 과정을 수행한다.

1. Parsing : HTML과 CSS를 파싱해서 각각 Tree(DOM 트리, CSSOM 트리)를 만든다
2. Style : 두 Tree를 결합하여 Rendering Tree를 만든다
3. Layout : Rendering Tree에서 각 노드의 위치와 크기를 계산한다 
4. Paint : 각 노드를 화면 상의 실제 픽셀로 변환하고 레이어를 만든다
5. Composite : 레이어를 합성하여 실제 화면에 나타낸다

<br/>

## html parsing 차단

브라우저가 html을 파싱하다가 `head` 태그 내 위치한 `script` 태그를 만났을 때 파싱이 차단되는 경우가 있다. 이는 `script`로 불러와지는 코드가 html 을 변경시킬 수 있기에 막는 것이다. 따라서 `script` 를 **`body` 태그 아래에 삽입**하여 html 파싱이 종료된 이후 로드하는 방법도 있지만, `script`의 **`async`, `defer` 속성**을 사용할 수 있다.

- `async` : 로드할 때는 차단하지 않음. 실행할땐 차단함
- `defer` : 로드, 실행할 때 모두 차단하지 않음.

**CSS도 간접적으로 차단할 수 있다.**  script가 페이지 스타일에 영향을 줄 수 있기에 css 파싱이 완료되고 js를 실행하도록하여, 간접적으로 html 파싱이 차단되는 시간이 길어질 수 있다. ([링크](https://seongil-shin.github.io/posts/css-%EC%B5%9C%EC%A0%81%ED%99%94/#css-%EC%B5%9C%EC%A0%81%ED%99%94%EA%B0%80-%ED%95%84%EC%9A%94%ED%95%9C-%EC%9D%B4%EC%9C%A0))

![https://web-now-rbviiass9-calibreapp.vercel.app/_next/image?url=%2Fimages%2Fblog%2Fcss-performance%2Fparser-blocking-css.png&w=1920&q=75](https://seongil-shin.github.io/assets/img/2022-07-31-css%20%EC%B5%9C%EC%A0%81%ED%99%94/imageurl=%2Fimages%2Fblog%2Fcss-performance%2Fparser-blocking-css.png)

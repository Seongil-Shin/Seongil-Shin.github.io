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
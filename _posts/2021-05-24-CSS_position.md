---
title: CSS position 에 관하여
author: 신성일
date: 2021-05-24 21:38:44 +0900
categories: [front end, CSS]
tags: [CSS, front end, position]
---

## static

- 기본 속성
- 원래 있어야할 위치에 있다.
- top, left, right, bottom 으로 위치 조절 불가능

## relative

- static 일 때의 위치를 기준으로 조절 가능.
- 겹치는 element 가 있으면, z-index로 결정한다.

## absolute

- position:static 을 가지고 있지 않은 부모를 기준으로 위치가 조절 된다.

## fixed

- 브라우저 기준으로 위치 조절 가능

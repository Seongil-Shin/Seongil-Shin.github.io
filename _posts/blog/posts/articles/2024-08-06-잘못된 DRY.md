---
title: 잘못된 DRY
author: 신성일
date: 2024-08-06 11:17:27 +0900
categories: study
tags:
  - "#study"
---
https://velog.io/@eunbinn/dry-the-common-source-of-bad-abstractions

- DRY 원칙을 너무 지키다보니 잘못된 추상화를 하는 경우가 있다
- 바로 서로 성격이 다른 코드이지만, 비슷한 패턴을 가진다는 이유로 같은 코드로 묶는 경우이다.
- ex) 네비게이션에 `Article`, `Menu`, `Buy` 버튼이 있을때, `Article`, `Menu`와 `Buy` 버튼은 서로 성격이 다르지만 같은 팩터리 메서드로 묶는다면, 추후 `Buy` 버튼만 변경될 우려가 있다.
- 따라서 서로 다른 성격의 코드는 다르게 묶거나, 다양한 형태에 대응 가능하도록 하는 것이 좋다.



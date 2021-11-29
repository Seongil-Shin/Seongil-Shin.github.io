---
title: LocateC
author: 신성일
date: 2021-11-25 18:19:26 +0900
categories: [projects]
tags: [project, expo]
---

## 개발 계기

- 코로나 전에 커피를 마시면서 자주 길거리를 걸어다녔는데, 항상 다 마시고 남은 빈 통만 들고 한참을 쓰레기통을 찾아다녔던 기억이 있었다.
- 그런데 학과 내에서 학교 생활에 도움을 줄 서비스를 개발하는 공모전을 하였고, 위 기억을 되살려서 학교 내 흡연장소, 쓰레기통의 위치를 알려주는 서비스를 개발해보기로 하였다.

## 서비스 소개

- 교내에 위치한 흡연장소, 쓰레기통의 위치를 사진과 함께 확인 가능.
- 가장 가까운 곳에 위치한 흡연장소/쓰레기통을 안내
- 사용자는 관리자에게 서버에 등록되지않은 새로운 위치를 추가하도록 요청을 보낼 수 있음.
- 관리자는 별도의 웹페이지를 통해 사용자의 요청을 처리하고, 이미 등록된 리스트도 관리할 수 있음.
- web, android, ios 크로스 플랫폼 지원

## 서비스 링크

- [web](https://locatec-public.web.app/)

- [expo_app](https://expo.dev/@callmeshin75/st-algorithm)

### custom hook

- 웹에서의 반응형/ 웹, 앱에서의 다크모드 구현을 위해 처음으로 custom hook 을 구현하여 사용하였다.
- 일전엔 일일히 하드코딩을 통해 구현하였던 반응형을 custom hook으로 구현하니 훨씬 코드가 깔끔해졌다.

### 다크모드

- 처음으로 앱/웹에서 다크모드를 지원하게 되었다.
- 다크모드의 영향을 받는 컴포넌트들을 별도의 파일로 분류하여 관리하였다.

### patch-package

- 웹에서 사용한 지도라이브러리에 문제가 있어, 처음으로 모듈을 수정하여 사용할 수 있도록 해주는 patch-package 를 이용하여, 모듈을 직접 수정하였다.

## 어려웠던 부분

### typescript

- typescript를 배운지 그리 오래되지 않아, 타입오류를 해결하는 것에 어려움이 있었다.

### 크로스 플랫폼에서의 지도 라이브러리 사용

- 지도 라이브러리로 앱에서는 react-native-maps, 웹에서는 react-native-web-maps 라이브러리를 사용하였는다.

- 앱에서 사용한 react-native-maps는 높은 완성도를 가지고 있어 활용에 크게 문제는 없었다.

- 하지만, 웹에서 사용한 react-native-web-maps 은 관리가 되어있지 않은 오래된 라이브러리였기에 몇몇 컴포넌트가 지원이 되지 않는 문제들이 있었다.

- 이를 해결하기 위해 patch-package를 사용하여 모듈을 수정하여 사용하였다.

## 결과

- 이 서비스로 학과 내 공모전에 참가하여 2위를 수상하였다.

## UI UX

![img_1](/assets/img/2021-11-25-st-algorithm/Screenshot_1637750381.png)

![img_2](/assets/img/2021-11-25-st-algorithm/Screenshot_1637750872.png)

![img_3](/assets/img/2021-11-25-st-algorithm/Screenshot_1637751957.png)

![img_4](/assets/img/2021-11-25-st-algorithm/Screenshot_1637752084.png)

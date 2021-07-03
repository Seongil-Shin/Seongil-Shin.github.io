---
title: React native 기초
author: 신성일
date: 2021-07-03 19:30:44 +0900
categories: [application, front end]
tags: [React native]]
---

## 개발환경 만들기

- 필요한 것 : 안드로이드 스튜디오, expo, 안드로이드 emulator

1. npm install -g expo-cli

2. 안드로이드 스튜디오 설치 -> 설치 후 sdk도 설치

3. 안드로이드 스튜디오을 키고, configuration에 들어가서 AVD manager을 누르고, create virtual device를 눌러 가상머신 만들기

## 프로젝트 시작

1. expo init [프로젝트명]
2. expo start 하면 실행됨
3. 아까 만든 가상머신을 실행시키고, run on android device를 누름
4. 가상머신 안에 있는 expo를 실행시키고, 주소를 카피하면 실행됨.
5. crtl + m 으로 개발자메뉴 들어갈 수 있음

## react-native 작동방식

- 자바스크립트로, ios 또는 android 안에 있는 자바스크립트 엔진을 통해 실행됨
- 이때 브릿지를 통해 명령을 전달하는데, react-native 컴포넌트들이 브릿지들이다.
- 이 브릿지로 많은 데이터를 보내게 되면 부하가 걸리게 됨. 그래서 인스타그램 같은 앱을 만들기엔 적합하지만, 3d video 게임 같은 것은 부적합하다.
- CSS
  - StyleSheet.create() 를 사용하여 css를 적용할 수 있음.
  - 하지만 웹에서처럼 작동하지는 않음 ex) 부모의 color를 받아오지 않음 등등

## React native layout

- 리액트네이티브의 flexDirection의 디폴트는 colum
- flex: 1- 모든공간이 사용가능함을 의미
  - 자식들의 flex를 조절하면서 각 자식들이 차지할 공간을 조절하면 됨.

## 개발

- 리액트 개발하는 방식대로 하면 된다.
- css는 StyleSheet 사용해서 만들고, 기능들은 expo 페이지에 들어가서 라이브러리로 설치하면 됨.
- gradient
  - 웹에서는 쉽게 만들 수 있지만, react-native에서는 다르게 해줘야함.
  - ex ) expo install expo-linear-gradient 같은 라이브러리를 설치하여 함.

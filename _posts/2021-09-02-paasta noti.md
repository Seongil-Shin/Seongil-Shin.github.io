---
title: expo push notification
author: 신성일
date: 2021-09-02 16:00:00 +0900
categories: [study, react-native]
tags: [expo]
---

## paasta notification 회의

- 방법들
  1. expo 서버로 보내는 방법 (https://docs.expo.dev/push-notifications/sending-notifications/)
     - expo에서 제공하는 라이브러리를 사용하면 됨.
     - but 자바 sdk를 제공하긴 하는데 이건 maven 기반이고, 현재 우리 프로젝트는 gradle 기반임
  2. fcm, apns 에 직접 보내는 방법 (https://docs.expo.dev/push-notifications/sending-notifications-custom/)
     - 두가지 코드를 써야함
     - 애플 개발자 계정 필요
  3. fcm에 apns를 연동하고, 2번에서 fcm으로만 보내는 방법
     - 공식 방법이 아니라 제대로 동작할지 의문.
     - 한번 테스트가 필요함
     - 애플 개발자 계정이 필요
  4. expo-notification 라이브러리를 포기하고, plain react-native 방법으로 noti 구현하기
     - 프론트쪽 코스트가 크게 발생함.
     - 불가능한 건 아닌데 기능에 비해 코스트가 너무 큼. notification과 연관없는 부분까지 영향을 끼치게 되서
     - 방법
       1. expo bare workflow - native 코드를 들어내는 작업
          - expo run:ios : xcode 필요해서 맥에서 해야함
          - 프로젝트가 bare 모드로 바뀌어서 나만이 아니라 다른 팀원들과 다른 코드들도 영향을 받게됨.
          - 설정하나하나를 직접 해줘야하는 코스트가 발생하게 되어 사실상 expo를 사용하는 이유가 줄어듦
       2. react-native-firebase 사용
          - ios 개발자계정 필요
          - 메세지 기능만 사용하기엔 너무 거대한 라이브러리.

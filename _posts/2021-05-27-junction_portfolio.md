---
title: Junctions/Seoul 2021 해커톤
author: 신성일
date: 2021-05-26 21:38:44 +0900
categories: [projects]
tags: []
---

## Junctions/Seoul 2021 해커톤

- Junction 은 핀란드에서 시작된 국제 해커톤으로, 2박 3일동안 진행된다.
- Autocrypto, Microsoft, SIA, AWS game tech 4개의 기업이 파트너로 참가하고, 참가자들은 이 4개의 기업 중 하나를 자신이 참가할 track으로 선택하여 참여한다.
- 수상은 track 별로 1,2,3등과 전체 우승으로 나뉜다. track 별 수상자는 기업의 관련자가 뽑으며, 전체 우승은 참가자들의 투표로 결정된다.

## 역할 및 결과

- 우리팀 Hippy는 AWS game tech track을 선택하였고, 나는 프론트엔트 뷰 역할을 맡았다.
- 좋은 팀원들과, 모두의 노력 끝에 track 1등을 차지하였다.

![image-20210524174313618](/assets/img/KakaoTalk_20210523_193727337.png)

## 뷰 화면 링크 (시연영상)

https://drive.google.com/file/d/10euUsveJwcJRgnblyzvXjSq3CRyzONDm/view?usp=sharing

## 배운점

- 디자이너분들과 협력하는 경험을 쌓았다.
  - 디자이너분들은 우리가 보지 못하는 세세한 부분까지 캐치가 가능하며, 사소한 모양 하나하나에 디자인 컨셉이 담겨있어 그 부분을 사소하다고 무시하면 안된다는 것을 깨달았다.
  - 기술적으로 혹은 cost 문제로 구현이 힘든 부분들은, 같이 토의를 하며 문제를 해결해나가는 경험을 쌓았다.
- lottie animation, figma, trello 등 툴들의 사용법을 익혔다.
- 처음으로 MVP 모델을 적용하여 컨셉을 이해하였다.

-----------이하 서비스 소개

## 서비스 소개

- 보이스 채팅 기능이 제공되지 않는 게임에 보이스채팅 기능을 제공하는 웹 서비스.
- 사용자는 웹 페이지에 접속 후, 플레이 하고자 하는 게임을 클릭 후, 닉네임을 입력하면 된다. 방에 입장 후 기다리면, 같은 게임 방에 들어온 사람들끼리 자동적으로 연결되도록 한다.

## 구현

- 게임을 접속했는지 확인하고, 같은 팀원들끼리 묶는 데는 게임사에서 제공하는 API를 통해 구현하였다.
- 이후 보이스채팅을 연결하는 것은, AWS Chime 을 사용하여 소켓통신을 통해 구현하였다.

---
title: 브라우저 저장소
author: 신성일
date: 2023-05-31  22:00:00 +0900
categories: [study, web]
tags: [browser]
---



브라우저에서 웹페이지를 위해 제공하는 저장소과 특징은 다음과 같다.

- Localstorage
  - 직접 지울 때까지 남아있다.
  - 같은 도메인은 같은 localstorage를 가짐
- SessionStorage
  - session 기간에만 데이터를 저장함.(보안에 유리). 
  - 같은 도메인을 여러개 열어도 다른 탭에 있으면 별도의 storage를 가짐
- Cookie
  - 서버에서 set-cookie를 통해 저장한 값. (js로도 값 접근/저장 가능) 
  - credential 옵션을 주면 API 요청 시에 헤더에 담겨 보내진다.
  - 옵션
    - httpOnly : js로는 접근 불가. 
    - secure : https 통신에서만 사용가능
    - expire : 만료일을 정해 스스로 사라지게 설정가능함.
    - path : 특정 path에서만 사용가능하게 설정
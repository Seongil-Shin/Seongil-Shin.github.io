---
title: WAS vs 웹서버
author: 신성일
date: 2022-09-12 20:00:00 +0900
categories: [study, web]
tags: [was, server]
---

## **WAS(Web Application Server)**

-  DB 조회나 다양한 로직처리를 요구하는 **동적인 컨텐츠**를 제공하기 위해 만들어진 Application Server
-  HTTP를 통해 컴퓨터나 장치에 애플리케이션을 수행해주는 미들웨어(소프트웨어 엔진)이다.
-  `웹 컨테이너` 혹은 `서블릿 컨테이너` 라고도 불린다.
   -  컨테이너란, jsp, servlet을 실행시킬 수 있는 소프트웨어를 말함
   -  즉, WAS란 JSP, servlet 구동환경을 제공한다.
-  쉬운 설명 : 포트에 연결되어 요청을 프로그램으로 쏴주고, 응답을 보내주는 미들웨어.
-  예시 : Tomcat, JBoss, Jeus, Web Sphere

**역할**

-  WAS = Web Server + Web container
-  웹 서블릿 기능들을 구조적으로 분리하여 처리하고자 하는 목적으로 제시되었다.
   -  분산 트랜잭션, 보안, 메시징, 스레드 처리등의 기능을 처리하는 분산환경에서 사용된다.
   -  주로 DB 서버와 함께 수행된다.
-  현재는 WAS가 가지고 있는 Web server도 정적인 컨텐츠를 처리하는데 있어서 성능상 큰차이가 없다.

**주요 기능**

-  프로그램 실행환경과 DB 접속 기능 제공
-  여러 개의 트랜잭션 관리기능
-  업무를 처리하는 비즈니스 로직 수행

<Br/>

## **Web server**

-  웹 브라우저 클라이언트로부터 HTTP 요청을 받아 **정적인 컨텐츠**를 제공하는 컴퓨터 프로그램
-  예시 : Apache Server, Nginx, IIS

**기능**

-  HTTP 프로토콜을 기반으로 하여 클라이언트의 요청을 서비스하는 기능을 담당한다.
-  기능 1)
   -  정적인 컨텐츠 제공
   -  WAS를 거치지 않고 바로 자원을 제공한다.
-  기능 2)
   -  동적인 컨텐츠 제공을 위한 요청전달
   -  클라이언트의 요청을 WAS에 보내고, WAS가 처리한 결과를 클라이언트에게 전달한다.

<br/>

## WAS와 웹서버를 분리한 이유

1. 기능을 분리하여 서버부하 방지
   -  단순한 정적 컨텐츠는 Web Server에서 빠르게 클라이언트에 제공하는 것이 좋기에.
2. 물리적으로 분리하여 보안 강화
   -  SSL에 대한 암복호화 처리에 웹서버 사용
3. 여러 대의 WAS 연결 가능
   -  로드 밸런싱을 위해 웹서버를 사용하고, 여러 개의 WAS를 웹서버에 붙일 수 있다.
   -  fail over, fail back 처리에 유리하다
   -  대용량 웹 어플리케이션(여러 개의 서버 사용)의 경우 웹서버와 WAS를 분리하여 무중단 운영을 위한 장애 극복에 쉽게 대응할 수 있다.
      -  ex) 오류가 발생한 WAS를 이용하지 못하도록, 앞 단의 웹 서버에서 처리할 수 있음. 그틈에 오류가 난 WAS를 재시작함으로써 사용자는 오류를 느끼지 못함

즉, 자원 이용의 효율성 및 장애극복, 배포 및 유지보수의 편의성을 위해 WAS와 웹서버를 분리한다.

## <Br/>

## **웹서버 + WAS 아키텍처**

## ![img](/assets/img/2022-09-12-was-webserver/web-service-architecture.png)

1. 웹 서버는 웹 브라우저 클라이언트로부터 HTTP 요청을 받는다.
2. 웹 서버는 클라이언트의 요청을 WAS에 보낸다.
3. WAS는 관련된 Servlet을 메모리에 올린다.
4. WAS는 web.xml을 참조하여 해당 Servlet에 대한 Thread를 생성한다.(Thread Pool 이용)
5. HttpServletRequest와 HttpServletResponse 객체를 생성하여 Servlet에 전달한다
   1. Thread는 Servlet의 service() 메서드를 호출한다
   2. service() 메서드는 요청에 맞게 doGet() 또는 doPost() 메서드를 호출한다.
   3. `protected doGet(HttpServletRequest request, HttpServletResponse response)`
6. doGet() 또는 doPost() 메서드는 인자에 맞게 생성된 적절한 동적 페이지를 Response 객체에 담아 WAS에 전달한다.
7. WAS는 Response 객체를 HttpResponse 형태로 바꾸어 Web Server에 전달한다
8. 생성된 Thread를 종료하고, HttpServletRequest와 HttpServletResponse 객체를 제거한다.

<br/>

## **여담**

node.js의 경우 엄밀히 말하면 `런타임`이지 웹 서버는 아니다. `웹서버를 가진 런타임`이라고 보는 것이 타당하다.

기능으로는 정정파일 제공, WAS 기능 모두 수행한다.

<Br/>

## **출처**

https://gmlwjd9405.github.io/2018/10/27/webserver-vs-was.html

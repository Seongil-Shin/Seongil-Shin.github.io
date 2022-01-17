---
title : RESTful API
author : 신성일
date : 2022-01-16 23:00:00 +0900
categories : [cs, basic]
tags: [cs]
---

`REST`란, REprensentational State Transfer의 약자이다. 따라서, RESTful API는 REST의 기본 원칙을 잘 지킨 API를 의미한다.

REST는 하나의 아키텍쳐로, Resource Oriented Architecture이다. API 설계의 중심에 자원(resource)가 있고, HTTP Method를 통해 자원을 처리하도록 설계하는 것이다.

> www와 같은 분산 하이퍼 미디어 시스템을 위한 소프트웨어 아키텍처의 한 형식으로, 자원을 정의하고 자원에 대한 주소를 지정하는 방법 전반에 대한 패턴



## REST 특징

1. uniform interface
   - URI로 지정한 리소스에 대한 조작을 통일되고 한정적인 인터페이스로 수행하는 아키텍쳐 스타일
2. Stateless
   - API 서버는 작업을 위한 상태정보를 따로 저장하지 않고, 들어온 요청을 단순히 처리하면 된다.
   - 때문에 서비스의 자유도가 높아지고 서버에서 불필요한 정보를 관리하지 않아도 된다.
3. caching
   - HTTP라는 기존 웹표준을 그대로 사용하기에, 웹에서 사용하는 기존 인프라를 그대로 사용할 수 있다.
   - 따라서 HTTP가 가진 캐싱 기능을 적용할 수 있다.
   - HTTP 프로토콜 표준에서 사용하는 Last-Modified 태그나 E-Tag를 이용하면 캐싱 구현이 가능하다/
4. client-server
   - REST 서버는 API를 제공하고, 클라이언트는 사용자 인증이나 컨텍스트(세선, 로그인정보) 등을 직접 관리하는 구조로, 각각의 역할이 확실히 구분되어있다.
   - 따라서 클라이언트와 서버에서 개발해야할 내용이 명확해지고, 서로간의 의존성이 줄어든다. 
5. hierarchical system
   - REST 서버는 다중 계층으로 구성될 수 있으며, 보안/로드밸런싱/암호화 계층을 추가해 구조상의 유연성을 둘 수 있고, Proxy, 게이트웨이 같은 네이워크 기반의 중간 매체를 사용할 수 있게 한다.
6. Self-descriptiveness
   - REST API 메시지만 보고도 이를 쉽게 이해할 수 있는 자체 표현구조로 되어있다.



## RESTfull 하게 API를 디자인 한다는 것의 의미

1. **리소스**와 **행위**를 명시적이고 직관적으로 분리한다.

   - 리소스는 URI로 표현되는데, 리소스가 가리키는 것은 명사로 표햔되어야한다.
   - 행위는 HTTP Method로 표현하고, `GET, POST, PUT(전체 수정), PATCH(일부 수정), DELETE` 가 있다.

2. Message는 Header와 Body를 명확하게 분리해서 사용한다

   - entity에 대한 내용은 body에 담는다
   - 애플리케이션 서버가 행동할 판단의 근거가 되는 컨트롤 정보인 API 버전 정보, 응답받고자 하는 MIME 타입등은 header에 담는다.
   - header와 body는 http header와 http body로 나눌 수 있고, http body에 들어가는 json 구조로 불리할 수 있다.

3. API 버전을 관리한다.

   - 환경은 항상 변하기때문에 API의 signature가 변경될 수도 있음에 유의하자.
   - 특정 API를 변경할 때는 반드시 하위호환성을 보장해야한다.

4. 서버와 클라이언트가 같은 방식을 사용해서 요청하도록 한다.

   - 브라우저는 form-data 형식의 submit으로 보내고 서버에서는 json형태로 보내는 식의 분리보다는, json으로 보내든, 둘다 form-data 형식으로 보내든 하나로 통일한다.

   - 다른 말로 표현하자면, URI가 플랫폼 중립적이어야한다는 말이다.



## 장단점

### 장점 

- Open API를 제공하기 쉽다.
- 멀티플랫폼 지원 및 연동이 용이하다.
- 원하는 타입으로 데이터를 주고 받을 수 있다.
- 기존 웹 인프라(HTTP)를 그대로 사용할 수 있다.

### 단점

- 사용할 수 있는 메서드가 적다.
- 분산환경에는 부적합하다.
- HTTP 통신 모델에 대해서만 지원한다.



## 출처

https://meetup.toast.com/posts/92


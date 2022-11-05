---
title: [spring] HTTP 메시지 컨버터
author: 신성일
date: 2022-11-05 18:34:26 +0900
categories: [study, spring]
tags: [spring, http, converter]
---

# HTTP 메시지 컨버터

스프링에서 컨트롤러를 개발하다보면, url 파라미터를 `long` 으로 받아도 문제없이 작동된다.

```java
@GetMapping(path = "/article/{articleId}")
public String getArticleDetail(@PathVariable long articleId) {
```

위와 같은 컨트롤러가 있을 때, 유저는 `/article/123`라는 요청을 보내면 자동으로 `getArticleDetail`의 매개변수인 `articleId`에 `123` 값이 문제없이 할당된다.

URL 파라미터는 기본적으로 문자열로 인식한다. 하지만 long에 할당이 일어나는 것은 무언가가 중간에서 문자열 `123`을 long 타입 `123`으로 변환시켜주는 것이다. 그 무언가가 스프링 MVC에서는 `HTTP 메시지 컨버터`이다.

스프링 MVC는 다음의 경우에 HTTP 메시지 컨버터를 적용한다

> HTTP 요청 : @RequestBody, HttpEntity(RequestEntity)
>
> HTTP 응답 : @ResponseBody, HttpEntity(ResponseEntity)

## 요청 매핑

HttpMessageConverter는 HTTP 메시지 컨버터의 인터페이스로 다음과 같은 메서드를 포함한다

-  canRead(), canWrite() : 메시지 컨버터가 해당 클래스, 미디어타입을 지원하는지 체크
-  read(), write() : 메시지 컨버터를 통해서 메시지를 읽고 쓰는 기능

스프링 부트는 다양한 메시지 컨버터를 제공하는데 다음과 같은 메시지 컨버터가 주요하며 적용 우선순위는 나열 순서와 같다

1. ByteArrayHttpMessageConverter : byte[] 데이터를 처리한다
   -  클래스 타입 : byte[] / 미디어타입 : \*/\*
2. StringHttpMessageConverter : String 데이터를 처리한다
   -  클래스타입 : String, 미디어타입 : \*/\*
3. MappingJackson2HttpMessageConverter : application/json 바디처리
   -  클래스타입 : 객체 또는 HashMap. 미디어타입 : application/json

<br/>

## 동작순서

메시지 컨버터가 동작하는 순서는 다음과 같다.

**HTTP 요청**

1. HTTP 요청이 오고, 컨트롤러에서 @RequestBody, HttpEntity 파라미터를 사용한다
2. 메시지 컨버터가 `canRead()`를 호출하여 메시지를 읽을 수 있는지 확인한다.
   -  대상 클래스 타입을 지원하는가, HTTP 요청의 Content-type 미디어 타입을 지원하는가
3. 읽을 수 있으면 `read()를 호출해서 객체를 생성하고 반환한다.

**HTTP 응답**

1. 컨트롤러에서 @ResponseBody, HttpEntity로 값이 반환된다.
2. 메시지컨버터가 메시지를 쓸 수 있는지 확인하기 위해 `canWrite()₩를 호출한다.
   -  대상 클래스 타입을 지원하는가, HTTP 요청의 Accept 미디어 타입을 지원하는가
3. 쓸 수 있으면 `write()`를 호출해서 HTTP 응답 메시지 바디에 데이터를 생성한다.

<br/>

## Argument Resolver, ReturnValueHandler

Http 메시지 컨버터는 언제 사용될까? 이것을 알아보기저에 먼저 `Argument Resolver`와 `ReturnValueHandler`를 알아보자.

애노테이션 기반의 컨트롤러(`@RequestMapping`을 사용하는 컨트롤러)를 처리하는 핸들러 어댑터는 `RequestMappingHandlerAdapter`이다. 이 RequestMappingHandlerAdapter는 핸들러 사이에 `Argument Resolver`와 `ReturnValueHandler`를 다음 그림과 같이 둔다.

![RequestMappingHandlerAdapter](https://velog.velcdn.com/images%2Fwoo00oo%2Fpost%2F4336ed57-d26d-4cd3-b985-c2c6ccc536ad%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-06-22%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%2011.45.08.png)

**ArgumentResolver**

애노테이션 기반의 컨트롤러는 매우 다양한 파라미터를 사용할 수 있다. `HttpServletRequest`, `Model`, `@RequestPaaram`, `@ModelAttribute` 등과 같은 애노테이션과 `@RequestBody`, `HttpEntity`와 같은 HTTP 메시지도 처리해준다. 이렇게 유연하게 처리해주는 것이 바로 ArgumentResolver이다.

애노테이션 기반 컨트롤러를 처리하는 RequestMappingHandlerAdapter는 이 ArgumentResolver를 호출하여 컨트롤러가 필요로하는 다양한 파라미터의 값을 생성한다.

스프링은 30개가 넘는 ArgumentResolver를 제공한다. 이 ArgumentResolver를 통해 각 파라미터를 처리하는 것이다.

[가능한 파라미터 목록](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-arguments)

**ReturnValueHandler**

ArgumentResolver와 마찬가지로 컨트롤러에서 반환되는 다양한 값을 변환하여 `ModelAndView` 객체로 만들어 RequestMappingHandlerAdapter에게 전달하는 역할을 맡는다.

[가능한 응답값 목록](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-return-types)

<br/>

## HTTP 메시지 컨버터

위와 같은 구조에서 Http 메시지 컨버터는 Argumen Resolver와 ReturnValueHandler가 호출될때 사용된다.

![HTTP 메시지 컨버터 위치](https://blog.kakaocdn.net/dn/bMdtRi/btrdmCHdq2E/gfUV8gRPgudfdhgCrKa58k/img.png)

**요청**

-  @RequestBody를 처리하는 ArgumentResolver와 HttpEntity를 처리하는 ArgumentResolver가 있다.
-  이 ArgumentResolver들이 HTTP 메시지 컨버터를 사용해서 필요한 객체를 생성한다.

**응답**

-  @ResponseBody와 HttpEntity를 처리하는 ReturnValueHandler가 있다.
-  이 ReturnValueHandler가 HTTP 메시지 컨버터를 사용하여 응답 결과를 만든다.

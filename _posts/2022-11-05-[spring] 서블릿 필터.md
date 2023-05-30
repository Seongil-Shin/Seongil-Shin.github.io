---
title: 서블릿 필터
author: 신성일
date: 2022-11-05 18:11:26 +0900
categories: [study, spring]
tags: [spring]
---

# 서블릿 필터

스프링에서 공통 관심사를 처리하기 위한 방법으로, 스프링 인터셉터와 함께 쓰이는 방법이다. 스프링 AOP를 사용하여 공통관심사를 처리할 수 있지만 웹과 관련된 공통 관심사는 서블릿 필터나 스프링 인터셉터를 사용하는 것이 좋다. 서플릿 필터나 스프링 인터셉터는 `HttpServletRequest`를 제공해주기 때문이다.

## 소개

**필터 흐름**

> HTTP 요청 -> WAS -> 필터 -> 디스패처 서블릿 -> 컨트롤러

필터는 디스패처 서블릿이 호출되기 전에 호출된다. 필터를 통해 다음과 같은 것들을 할 수 있다.

-  필터 제한 : 적절하지 않은 요청을 차단할 수 있다.

-  필터 체인 : 다음과 같이 필터를 연속으로 적용하여 체인처럼 사용할 수 있다.

   > HTTP 요청 -> WAS -> 필터1 -> 필터2 -> 필터3 -> 디스패처 서블릿

**필터 인터페이스**

필터인터페이스는 다음과 같은 메서드가 등록되어있다. 이 필터 인터페이스를 구현하고 등록하면 `서블릿 컨테이너`가 필터를 싱글톤 객체로 생성하고 관리한다.

-  Init() : 필터 초기화 메서드. 서블릿 컨테이너가 생성될 떄 호출된다.
-  doFilter() : 고객의 요청이 올 때마다 해당 메서드가 호출된다. 필터의 로직을 구현하면 된다.
   -  return; 하면 디스패처 서블릿으로 진행하지 않음.
-  destroy() : 필터 종료 메서드. 서블릿 컨테이너가 종료될 때 호출된다.

**필터 등록**

스프링 부트에서는 다음과 같이 `FilterRegistrationBean`를 사용하여 등록하면 된다. 이때 등록하는 메서드는 `@Configuration` 로 등록한 클래스 아래 위치해야한다.

```java
@Configuration
public class WebConfig {
  @Bean
  public FilterRegistrationBean logFilter() {
  	 FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();
    filterRegistrationBean.setFilter(new LogFilter());
    filterRegistrationBean.setOrder(1);
    filterRegistrationBean.addUrlPatterns("/*");
    return filterRegistrationBean;
  }
}
```

-  setFilter : 등록할 필터를 지정한다.
-  setOrder : 필터가 적용되는 순서를 정함. 낮은 숫자가 먼저 실행된다.
-  addUrlPatterns() : 필터를 적용할 URL 패턴을 지정한다. 한번에 여러 패턴을 지정할 수 있다.
   -  이때 적용되는 패턴은 `서블릿 URL 패턴`이라고 검색하면 된다.

이외 `@ServletComponentScan`, `@WebFilter(filterName="logFilter", urlPatterns="/*")`으로 필터 등록이 가능하지만, 필터 순서 조절이 안된다. 따라서 `FilterRegistrationBean`을 사용하는 것이 좋다.

## 요약
서블릿 필터는 스프링 인터셉터와 함께 웹 요청이 왔을 때 공통 관심사를 처리해주는 방법이다. 
서블릿 필터는 스프링 인터셉터와 달리 디스패처 서블릿 이전에 호출된다.
필터 인터페이스를 구현하고, 빈에 등록하여 서블릿 필터를 등록할 수 있다

## 출처

-  [김영한의 스프링 MVC 2편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-2/dashboard)

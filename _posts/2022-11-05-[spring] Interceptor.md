---
title: interceptor
author: 신성일
date: 2022-11-05 18:35:26 +0900
categories: [study, spring]
tags: [interceptor, spring]
---

# 스프링 인터셉터

스프링 인터셉터는 스프링 MVC가 제공하는 기술로, 서블릿 필터하고는 적용순서, 범위, 사용방법이 다르다.

<br/>

## 소개

**스프링 인터셉터 흐름**

> HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 -> 컨트롤러

-  스프링 인터셉터는 디스패처 서블릿과 컨트롤러 사이에서 컨트롤러 호출 직전에 호출됨.

-  스프링 인터셉터는 스프링 MVC가 제공하는 기능으로, 디스패처 서블릿 이후에 등장하게 된다. 스프링 MVC의 시작점이 디스패처 서블릿이라고 생각하면 됨.

-  스프링 인터셉터도 URL 패턴을 적용할 수 있다. 서블릿 URL 패턴과는 다르고 매우 정밀하게 설정가능

-  스프링 인터셉터에서 적절하지 않은 요청이라고 판단하면 거기서 요청을 차단할 수 있다. 컨트롤러까지 가지 않도록 한다.

-  다음과 같이 체인을 구성할 수도 있다.

   > 서블릿 -> 인터셉터1 -> 인터셉터2 -> 컨트롤러

-  인터셉터는 싱글톤처럼 사용된다. 따라서 멤버변수를 사용하여 메소드간 값을 공유하면 위험하다.

**스프링 인터셉터 인터페이스**

스프링 인터셉터를 사용하려면 `HandlerInterceptor` 인터페이스를 구현하면 된다. 해당 인터페이스에는 다음과 같이 세가지 메서드가 있다

-  preHandle : 컨트롤러 호출 전에 호출됨. true 반환 시 다음으로 진행, false 반환 시 컨트롤러 호출하지 않음.
-  postHandle : 컨트롤러 호출 후에 호출된다. preHandle에서 false 반환 시 호출되지 않음. 예외가 발생했을 때에서 호출되지 않음
-  afterCompletion : 뷰가 렌더링 된 다음에 호출됨. preHandle에서 false 가 반환되더라도 항상 호출됨.
   -  afterCompletion은 예외가 발생해도 호출된다. 따라서 공통 처리를 할 때는 afterCompletion을 사용한다.
   -  예외가 발생했을 때는 예외 정보(ex)를 포함해서 호출된다. (파라미터로 받음)

스프링 인터셉터의 메서드에서는 `request`, `response`, `handler` 정보를 받을 수 있다. handler는 호출되는 컨트롤러를 뜻한다.

**인터셉터 등록**

인터셉터는 WebMvcConfigurer에 다음과 같이 등록해줘야한다.

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
      @Override
      public void addInterceptors(InterceptorRegistry registry) {
          registry.addInterceptor(new LogInterceptor())
                  .order(1)
			            .addPathPatterns("/**")
									.excludePathPatterns("/css/**", "/*.ico", "/error");
      }
}
```

-  order : 인터셉터의 호출 순서를 지정. 낮을수록 먼저 호출된다.
-  addPathPatterns ; 인터셉터를 적용할 URL 패턴을 지정한다
-  excludePathPatterns : 인터셉터에서 제외할 패턴을 지정한다.

**PathPatterns**

스프링이 제공하는 URL 경로 기술로, 매우 자세하게 설정할 수 있다.

[path patterns 공식문서](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/util/pattern/PathPattern.html)

<br/>

## interceptor에서 controller로 값 전달하기

### **recommended : request 스코프의 빈 사용**

다음과 같이 전달할 값을 필드로 지닌 클래스를 만들고, request 스코프로 빈에 등록하는 것이 가장 좋다.

```java
@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class CurrentUser {
    private User currentUser;

    public User getCurrentUser() {
        return currentUser;
    }

    public void setCurrentUser(User currentUser) {
        this.currentUser = currentUser;
    }
}
```

### **alternative : HttpServletRequest에 추가**

필드가 딱 하나만 있거나, 컨트롤러에서도 HttpServletRequest를 사용한다면 다음과 같이 HttpServletRequest에 값을 추가하는 것도 방법이다.

```java
@Component
public class MyInterceptor implements HandlerInterceptor {
   @Override
   public boolean preHandle(
     HttpServletRequest request,
     HttpServletResponse response, Object handler) throws Exception {
      request.setAttribute("email", "login@domain.com");
      return true;
   }
}
```

[참고](https://stackoverflow.com/questions/58942591/spring-boot-pass-argument-from-interceptor-to-method-in-controller)

<br/>

## ExceptionHandler와의 조합

interceptor에서 날아간 예외는 어떻게 처리하는 것이 좋을까?

이에 관해선 [API 예외처리](https://seongil-shin.github.io/posts/spring-API-%EC%98%88%EC%99%B8%EC%B2%98%EB%A6%AC/) 문서의 **@ExceptionHandler, @ControllerAdvice 원리** 섹션에서 자세히 정리해놓았지만, 간단히 요약하자면 아래와 같다. 

- interceptor의 preHandler와 postHandle에서 던진 예외는 ExceptionHandler에서 잡을 수 있다.
  - `@ExceptionHandler`로 선언한 핸들러와 `@ControllerAdvice`로 등록한 핸들러 모두 적용됨
- afterCompletion에서 던진 예외서 ExceptionHandler에서 처리되지 않고 스프링에서 처리된다. 다만 이때 클라이언트 요청의 응답은 제대로 가고, 스프링 내부에서 예외가 처리된다.

<br/>

## 참고

-  [김영한의 스프링 MVC 2편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-2/dashboard)
-  [interceptor에서 controller로 값 전달하기](https://stackoverflow.com/questions/58942591/spring-boot-pass-argument-from-interceptor-to-method-in-controller)

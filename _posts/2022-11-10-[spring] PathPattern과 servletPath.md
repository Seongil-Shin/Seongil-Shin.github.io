---
title: PathPattern과 servletPath
author: 신성일
date: 2022-11-10 18:25:26 +0900
categories: [study, spring]
tags: [spring]
---

# **[spring] PathPattern과 servletPath**

## 문제

다음과 같이 interceptor를 등록할 때, `addPathPatterns()`에 `/api/**`를 등록하였지만, `/api/article`을 요청했을 때 인터셉터가 제대로 동작하지 않았다. 하지만 `/**`로 등록하니 제대로 동작하였다.

```java
public void addInterceptors(InterceptorRegistry registry) {
		registry.addInterceptor(authInterceptor)
			.order(1)
			.addPathPatterns("/api/**");		// not work
}

public void addInterceptors(InterceptorRegistry registry) {
		registry.addInterceptor(authInterceptor)
			.order(1)
			.addPathPatterns("/**");			// work
}
```

<br/>

## 단서

내 앱에서 `application.properties`에 `server.servlet.contextPath=/api` 로 모든 servlet의 prefix 주소를 `/api`로 설정한 것을 떠올렸다.

```properties
server.servlet.contextPath=/api
```

이 단서를 중심으로 찾아보니 pathPattern에 등록할 땐 전체 요청 주소에서 contextPath를 땐 servletPath를 사용해야한다는 것을 알게되었다.

만약 `/api/article`로 요청한다면 contextPath로 `/api`를 설정하였으므로 servletPath는 `/article`이 되어 `addPathPatterns()`에 등록한 `/api/**`이 작동하지 않았던 것이다.

<br/>

## **왜 servletPath를 써야할까?**

그렇다면 왜 스프링에서는 contextPath를 떼서 검사하는 것일까? 그것은 context path와 servlet path의 정의와 역할이 다르기 때문이다.

-  context path : 어플리케이션의 루트. 디폴트는 "/"
-  servlet path : prefix는 dispatcher servlet의 주소를 의미(디폴트는 "/"). prefix를 포함한 전체 주소는 스프링에서 처리되는 주소.

스프링 인터셉터를 등록하기위해 구현한 인터페이스는 `WebMvcConfigurer`이다. WebMvcConfigurer은 다음과 같이 DispatherServlet 이후에 스프링 MVC 설정을 위해 쓰인다.

![WebMvcConfigurer](https://4.bp.blogspot.com/-qy8ZQiputn0/WsTO_eK8FbI/AAAAAAAAMB4/YbueFWG54fEtY_DgJONuHH4sPjowjV6EgCLcBGAs/s1600/2.png)

인터셉터가 적용되거나(addPathPatterns로 등록된 주소), 적용되지 않는 주소(excludePathPatterns로 등록된 주소)를 검사하는 곳은 스프링 MVC이므로, 스프링에서 처리되는 주소인 Servlet Path가 검사 대상이 된다.

생각해보면 어플리케이션의 루트주소 이후부터 검사하는 것이 타당하긴 하다..

<br/>

## 출처

https://www.baeldung.com/spring-context-vs-servlet-path

https://lalwr.blogspot.com/2018/04/spring-annotation.html

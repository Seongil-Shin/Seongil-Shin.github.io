---
title: ArgumentResolver 활용
author: 신성일
date: 2022-11-05 18:31:26 +0900
categories: [study, spring]
tags: [spring, ArgumentResolver]
---

# ArgumentResolver 활용

ArgumentResolver는 스프링 MVC 구조의 어댑터 핸들러에서 핸들러에 필요한 파라미터를 만들어주는데 호출하는 부분이다. 이 ArgumentResolver를 직접 구현하여 활용하면 다양한 상황에서 편리하게 적용할 수 있다.

## 로그인 회원 정보를 편리하게 받아오는 예제

**컨트롤러**

```java
@GetMapping("/")
public String homeLoginV3ArgumentResolver(@Login Member loginMember, Model model)
```

**Login 어노테이션**

```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface Login {
}
```

-  @Target(ElementType.PARAMETER) : 파라미터에만 적용
-  @Retention(RetentionPolicy.RUNTIME) : 리플렉션 등을 활용할 수 있도록 런타임까지 애노테이션 정보가 남아있음.

**LoginMemberArgumentResolver**

`HandlerMethodArgumentResolver`를 구현하여 만든다.

```java
public class LoginMemberArgumentResolver implements HandlerMethodArgumentResolver {
	@Override
	public boolean supportsParameter(MethodParameter parameter) {
		boolean hasLoginAnnotation = parameter.hasParameterAnnotation(Login.class);
    boolean hasMemberType = Member.class.isAssignableFrom(parameter.getParameterType());
    return hasLoginAnnotation && hasMemberType;
  }

	@Override
  public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer,
                                NativeWebRequest webRequest,
                                WebDataBinderFactory binderFactory) throws Exception {
    	HttpServletRequest request = (HttpServletRequest)webRequest.getNativeRequest();
      HttpSession session = request.getSession(false);
      if (session == null) {
      	return null;
      }
      return session.getAttribute(SessionConst.LOGIN_MEMBER);
  }
}
```

-  supportsParameter : `@Login` 애노테이션이 있으면서 `Member` 타입이면 해당 `ArgumentResolver`가 사용되도록 적용
-  resolveArgument : 컨트롤러 호출 직전에 호출되어 필요한 파라미터 정보를 생성해준다.

**WebMvcConfigurer에 설정 추가**

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
	@Override
  public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
  	resolvers.add(new LoginMemberArgumentResolver());
  }
}
```

## 출처

-  [김영한의 스프링 MVC 2편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-2/dashboard)

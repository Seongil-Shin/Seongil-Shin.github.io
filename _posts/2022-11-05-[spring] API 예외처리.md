---
title: API 예외처리
author: 신성일
date: 2022-11-05 18:17:26 +0900
categories: [study, spring]
tags: [spring, api, exception]
---

# API 예외처리

## 예외 발생 시에도 JSON 응답 보내기

ResponseEntity를 사용해서 응답을 보내면 된다.

```java
public ResponseEntity<String> errorPage500Api() {
	String result = "error";
	return new ResponseEntity(result, HttpStatus.BAD_REQUEST.value());
}
```

ResponseEntity를 사용해서 응답하기 때문세 메시지 컨버터가 동작하면서 클라이언트에 JSON이 반환된다.

<br/>

## 스프링부트 예외 처리

### **BasicErrorController**

BasicErrorController은 스프링부트에서 기본으로 오류를 처리해주는 컨트롤러이다. 따로 등록한 것이 없다면 스프링부트는 예외가 컨트롤러 밖으로 예외가 던져진 경우 `/error`를 호출한다. 그리고 `/error`는 BasicErrorController로 매핑되어 아래 메서드를 호출한다.

```java
@RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse
response) {}

@RequestMapping
public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {}
```

-  errorHtml : `produces=MediaType.TEXT_HTML_VALUE` 으로 매핑되어, 클라이언트 요청의 Accept 헤더 값이 `text/html`인 경우 호출하여 에러 view를 제공한다.
-  error : 그 외의 경우 호출되며 ResponseEntity로 HTTP Body에 JSON 데이터를 반환한다.

error 발생 시 스프링부트가 ResponeEntity body에 넣어주는 정보는 다음과 같다

-  timestamp : 발생 시각
-  path : 클라이언트 요청 경로
-  status : 에러 상태
-  message : 에러 메세지
-  error : 에러 코드
-  exception : 발생한 예외
-  errors : 발생한 에러
-  trace : 예외 trace

application.properties에서 다음과 같이 응답에 정보를 담을지 말지 선택할 수 있다.

-  `server.error.include-exception`
-  `server.error.include-message`
-  `server.error.include-stacktrace`
-  `server.error.include-binding-errors`

BasicErrorController은 HTML 에러 페이지지를 제공할 때는 매우 편리하다. 하지만 API 오류를 내보낼 때는 컨트롤러마다 다른 오류 처리 방법에 대응하기가 어렵다. 따라서 API 오류를 처리할 때는 `@ExceptionHandler`를 사용하는 것이 좋다.

<br/>

### **HandlerExceptionResolver**

스프링 MVC는 컨트롤러 밖으로 예외가 던져진 경우, 이 예외를 잡고 처리하는 HandlerExceptionResolver를 제공해준다.

![image-20221103143127595](/Users/user/Library/CloudStorage/OneDrive-개인/study/assets/img/Monosnap 9. API 예외 처리.pdf 2022-11-03 16-08-16.png)

이때 HandlerExceptionResolver는 예외를 해결해도 postHandle()은 호출되지 않는다.

**HandlerExceptionResolver 인터페이스**

```java
public interface HandlerExceptionResolver {
	ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response,
		Object handler, Exception ex);
}
```

-  resolveException이 ModelAndView를 반환하는 이유는 Exception을 처리해서 정상 흐름처럼 변경하는 것이 목적이기 때문이다.

**HandlerExceptionResolver 구현**

```java
@Slf4j
public class MyHandlerExceptionResolver implements HandlerExceptionResolver {
	@Override
  public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
		try {
			if (ex instanceof IllegalArgumentException) {
				response.sendError(HttpServletResponse.SC_BAD_REQUEST, ex.getMessage());
				return new ModelAndView();
			}
		} catch (IOException e) {
			log.error("resolver ex", e);
    }
		return null;
  }
}
```

**resolveException 반환 값에 따른 동작방식**

-  빈 ModelAndVIew : `new ModelAndView()`를 반환하는 경우. 뷰를 렌더링하지 않고, 정상 흐름으로 서블릿이 리턴된다. 즉, API 오류 처리를 할 때 사용한다.
-  ModelAndVIew 지정 : ModelAndVIew에 View, Model 등의 정보를 지정해서 반환하면 뷰를 렌더링한다.
-  null : 다음 ExceptionResolver를 찾아서 실행한다. 다음이 없으면 예외처리가 안되고 기존에 발생한 예외를 서블릿 밖으로 던진다.

**ExceptionResolver 활용**

-  예외 상태 코드 변환
   -  예외를 response.sendError() 호출로 변경해서 서블릿에서 상태코드에 따른 오류를 처리하도록 위임
   -  이후 WAS는 서블릿 오류 페이지를 찾아서 내부 호출.
-  뷰 템플릿 처리
   -  ModelAndView에 값을 채워서 예외에 따른 새로운 오류화면 뷰 렌더링해서 고객에게 제공
-  API 응답처리
   -  response.getWriter().println() 처럼 HTTP응답 바디에 직접 데이터를 넣어주는 것도 가능. 여기에 JSON으로 응답하면 API 응답 처리를 할 수 있다.

**ExceptionResolver 등록**

-  WebMvcConfigurer의 extendHandlerExceptionResolvers를 통해 다음과 같이 등록하면 된다.

```java
@Override
public void extendHandlerExceptionResolvers(List<HandlerExceptionResolver>
resolvers) {
	resolvers.add(new MyHandlerExceptionResolver());
}
```

-  configureHandlerExceptionResolvers 로도 등록이 되지만, 스프링이 기본으로 등록하는 ExceptionResolver가 제거되므로 주의한다. extendHandlerExceptionResolvers는 확장.

<br/>

## 스프링이 제공하는 ExceptionResolver

스프링 부트가 기본으로 제공하는 ExceptionResolver와 우선순위는 다음과 같다.(HandlerExceptionResolverComposite에 다음과 같이 등록)

1. ExceptionHandlerExceptionResolver
   -  @ExceptionHandler를 처리한다.
2. ResponseStatusExceptionResolver
   -  HTTP 상태 코드를 지정해준다.
3. DefaultHandlerExceptionResolver
   -  스프링 내부 기본 예외를 처리한다.

<br/>

### ExceptionHandlerExceptionResolver

@ExceptionHandler 애노테이션을 사용한 예외처리 방법.

컨트롤러 안에서 @ExceptionHandler 와 처리하고 싶은 예외를 선언해주면 된다.

```java
@RestController
public class ApiExceptionV2Controller
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(IllegalArgumentException.class)
    public ErrorResult illegalExHandle(IllegalArgumentException e) {
        return new ErrorResult("BAD", e.getMessage());	//  @ResponseBody 적용됨
    }

    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    @ExceptionHandler
    public ResponseEntity<ErrorResult> exHandle(Exception e) {
      ErrorResult errorResult = new ErrorResult("EX", "내부 오류");
      return new ResponseEntity<>(errorResult, HttpStatus.BAD_REQUEST);
    }
  	...
}

```

예외가 발생하면 컨트롤러 안의 @ExceptionHandler 애노테이션이 붙은 메서드를 확인하고 예외를 처리한다. 지정한 예외와 그 예외의 자식 클래스는 모두 잡을 수 있다.

부모의 자식 예외가 모두 등록되어있을 때는, 자식 예외 처리가 먼저 호출된다.

다음과 같이 다양한 예외를 한번에 등록할 수도 있다.

```java
@ExceptionHandler({AException.class, BException.class})
```

다음과 같이 생략할 경우에는 메서드 파라미터의 예외로 지정된다.

```java
@ExceptionHandler
public ResponseEntity<ErrorResult> userExHandle(UserException e) {}
```

<br/>

### ResponseStatusExceptionResolver

예외에 따라서 HTTP 상태 코드를 지정해주는 역할을 한다. 다음 두 가지 경우를 처리한다.

-  @ResponseStatus가 달려있는 예외
-  ResponseStatusException 예외

**@ResponseStatus가 달려있는 예외**

다음과 같이 예외에 `@ResponseStatus` 애노테이션을 적용하면 HTTP 상태코드를 변경해준다.

```java
@ResponseStatus(code = HttpStatus.NOT_FOUND, reason = "찾을 수 없음")
public class NotFoundException extends RuntimeException {
}
```

NotFoundException 예외가 컨트롤러 밖으로 넘어가면 ResponseStatusExceptionResolver가 해당 애노테이션을 확인해서 오류 코드를 404로 변경하고 메시지를 담는다.

reason에는 MessageSource를 찾는 기능도 제공한다.

```java
@ResponseStatus(code = HttpStatus.NOT_FOUND, reason = "error.notfound")
```

```properties
# messages.properties
error.notfound=찾을 수 없습니다.
```

**ResponseStatusException**

@ResponseStatus는 개발자가 직접 변경할 수 없는 예외에는 적용할 수 없다. 이떄는 ResponseStatusException를 사용하면 된다.`

```java
@GetMapping("/api/response-status-ex2")
public String responseStatusEx2() {
	throw new ResponseStatusException(HttpStatus.NOT_FOUND, "error.notfound", new Exception());
}
```

<br/>

### DefaultHandlerExceptionResolver

스프링 내부에서 발생하는 스프링 예외를 해결한다.

-  ex) 파라미터 바인딩 실패시 발생하는 TypeMismatchException 예외를 처리하여 400으로 응답

<Br/>

## @ControllerAdvice

`@ExceptionHandler`를 컨트롤러 안에서 사용하여 예외를 잡을 수 있지만, 컨트롤러 내의 정상 코드와 예외 처리 코드가 섞여있다. 이를 분리하기 위해선 `@ControllerAdvice` 또는 `@RestControllerAdvice`를 사용하면 둘을 분리할 수 있다.

```java
@RestControllerAdvice
public class ExControllerAdvice {
      @ResponseStatus(HttpStatus.BAD_REQUEST)
      @ExceptionHandler(IllegalArgumentException.class)
      public ErrorResult illegalExHandle(IllegalArgumentException e) {
          return new ErrorResult("BAD", e.getMessage());
      }
  ...
}

```

@ControllerAdvice는 대상으로 지정한 여러 컨트롤러에 @ExceptionHandler, @InitBinder 기능을 부여해주는 역할을 한다. 대상을 지정하지 않으면 모든 컨트롤러에 적용한다.

```java
@ControllerAdvice(annotations = RestController.class)
public class ExampleAdvice1 {}


// Target all Controllers within specific packages
@ControllerAdvice("org.example.controllers")
public class ExampleAdvice2 {}

// Target all Controllers assignable to specific classes
@ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class})
public class ExampleAdvice3 {}
```

@RestControllerAdvice는 @ControllerAdvice와 같은 역할을 하며, 내부의 메서드에 @ResponseBody 애노테이션을 추가시켜준다.

<br/>

## 정리

HTML 에러 화면을 보여줘야하는 경우

-  BasicErrorController 사용

API 에러 응답을 보내야하는 경우

-  ExceptionHandlerExceptionResolver (@ExceptionHandler) 사용
-  @ControllerAdvise는 예외처리 코드와 정상코드를 분리시키고자 할 때 사용

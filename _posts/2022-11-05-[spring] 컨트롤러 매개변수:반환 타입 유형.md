---
title: 컨트롤러 매개변수/반환 타입 유형
author: 신성일
date: 2022-11-05 18:14:26 +0900
categories: [study, spring]
tags: [spring]
---

## **요청**

### 파라미터(쿼리, form 데이터)

**HttpServletRequest**

```java
@RequestMapping("/request-param-v1")
public void requestParamV1(HttpServletRequest request, HttpServletResponse response)
```

HttpServletRequest에서 조회해서 꺼내올 수 있다.

**@RequestParam**

```java
 @RequestMapping("/request-param-v2")
 public String requestParamV2(
            @RequestParam("username") String memberName,
            @RequestParam("age") int memberAge) {
```

이때 파라미터 이름과, 매개변수에 쓰인 변수명이 같다면 다음과 같이 생략할 수있다.

```java
@RequestMapping("/request-param-v3")
public String requestParamV3(
            @RequestParam String username,
            @RequestParam int age) {
```

또한 만약 String, int, Integer와 같이 단순 타입이면 @RequestParam을 생략할 수도 있다.

```java
@RequestMapping("/request-param-v4")
public String requestParamV4(String username, int age)
```

but 너무 없애면 과할 수도 있다는걸 생각하자.

번외로 다음과 같이 required 옵션을 주면 해당 파라미터가 없을 때 400 예외를 내도록 할 수 있다. defaultValue도 줄 수 있다.

```java
@RequestMapping("/request-param-required")
public String requestParamRequired(
          @RequestParam(required = true, defaultValue = "guest") String username,
          @RequestParam(required = false, defaultValue = "-1") int age)
```

주의

-  파라미터 이름만 사용 : `/request-param?username=` --> 이 경우는 빈문자로 통과한다.
-  primitive 타입에 null 입력 : `/request-param` --> 이 경우에는 required=false 로 지정된 매개변수엔 null이 들어가는데, primitive 타입엔 null 할당이 불가능하므로 500 에러가 난다.
-  defaultValue + 빈문자 : `/request-param?username=` --> 빈문자도 defaultValue가 적용된다.

다음과 같이 맵으로 조회하는 방법도 있다.

```java
@RequestMapping("/request-param-map")
public String requestParamMap(@RequestParam Map<String, Object> paramMap)

// "?value=첫번째&value=두번째" 처럼 여러개가 들어올 땐 MultiValueMap 사용
@RequestMapping("/request-param-map")
public String requestParamMap(@RequestParam MultiValueMap<String, Object> paramMap)
```

**@ModelAttribute**

파라미터를 객체의 형태로 받을 수 있도록 하는 애노테이션이다.

```java
@RequestMapping("/model-attribute-v1")
public String modelAttributeV1(@ModelAttribute HelloData helloData)
```

@ModelAttribute 동작원리

1. HelloData 객체를 생성한다.
2. 요청 파라미터의 이름으로 HelloData 객체의 프로퍼티를 찾는다. 그리고 해당 프로퍼티의 setter를 호출해서 값을 바인딩한다.

바인딩 오류

-  타입이 다르거나, primitive 타입에 null을 대입하는 등 바인딩 오류가 발생할 시 스프링 MVC는 `BindException`을 발생시킨다.

@ModelAttribute 는 다음과 같이 생략할 수 있다. 하지만, 너무 생략하면 이해하기 어려운 코드가 될 수 있다.

```java
@RequestMapping("/model-attribute-v2")
public String modelAttributeV2(HelloData helloData)
```

<br/>

### HTTP 요청 메시지 - 단순 텍스트

**HttpServletRequest**

```java
@PostMapping("/request-body-string-v1")
public void requestBodyString(HttpServletRequest request) {
  ServletInputStream inputStream = request.getInputStream();
  String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
}
```

HttpServletRequest를 통해 http request 메시지 전체를 받고, InputStream을 꺼내어 body를 만드는 방법이 있다. 많이 번거로운 방법.

**InputStream**

```java
@PostMapping("/request-body-string-v2")
public void requestBodyStringV2(InputStream inputStream) {
    String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
}
```

바로 InputStream을 꺼내는 방법도 가능하다.

**HttpEntity**

```java
  @PostMapping("/request-body-string-v3")
  public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) {
      String messageBody = httpEntity.getBody();
  }
```

HttpEntity로 body를 직접 꺼내올 수 있다. (header도 조회가능)

RequestEntity는 HttpEntity를 상소받아 만든 객체이다. 따라서 마찬가지로 body를 바로 가져올 수 있다.

**@RequestBody**

```java
@PostMapping("/request-body-string-v4")
public String requestBodyStringV4(@RequestBody String messageBody)
```

애노테이션 기반으로 바로 가져올 수도 있다.

<br/>

### HTTP 요청 메시지 - JSON

**HttpServletRequest**

```java
public void requestBodyJsonV1(HttpServletRequest request,
				HttpServletResponse response) throws IOException {
  ServletInputStream inputStream = request.getInputStream();
  String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

  ObjectMapper objectMapper = new ObjectMapper();
  HelloData data = objectMapper.readValue(messageBody, HelloData.class);
}
```

InputStream을 가져와 ObjectMapper로 변환시켜주면 된다. 매우 번거로움

**@RequestBody**

```java
@PostMapping("/request-body-json-v3")
public String requestBodyJsonV3(@RequestBody HelloData data)
```

애노테이션 기반으로 객체 형식으로 바로 가져올 수 있다.

이때 `@RequestBody` 애노테이션은 생략 불가능하다. 그 이유는 스프링에서는 `@ModelAttribute`, `@RequestParam`과 같은 애노테이션을 생략할 시 다음과 같은 규칙을 적용하기 때문이다.

1. String, int, Integer 같은 단순 타임 => @RequestParam 적용
2. 나머지 => @ModelAttribute 적용

따라서 @RequestBody를 생략하면 HelloData는 단순타입이 아니므로 **@ModelAttribute가 적용**되기에 @RequestBody는 생략해서는 안된다.

**HttpEntity**

```java
@PostMapping("/request-body-json-v4")
public String requestBodyJsonV4(HttpEntity<HelloData> httpEntity) {
	HelloData data = httpEntity.getBody();
}
```

<br/>

## 응답

**정적 리소스, 뷰 템플릿**

**ModelAndView**

```java
@RequestMapping("/response-view-v1")
public ModelAndView responseViewV1() {
	ModelAndView mav = new ModelAndView("response/hello").addObject("data", "hello!");
	return mav;
}
```

**String 반환**

```java
@RequestMapping("/response-view-v2")
public String responseViewV2(Model model) {
	model.addAttribute("data", "hello!!");
	return "response/hello";
}
```

**void 반환**

```java
@RequestMapping("/response/hello")
public void responseViewV3(Model model) {
	model.addAttribute("data", "hello!!");
}
```

요청 URL을 참고하여 논리 뷰 이름으로 사용한다.

> 요청 URL : /response/hello
>
> 실행 : templates/response/hello.html

하지만 이 방법은 가시성이 떨어지기에 웬만하면 다른 방법을 쓰자.

### **Body 응답**

**HttpServletResponse**

```java
public void responseBodyV1(HttpServletResponse response) throws IOException {
	response.getWriter().write("ok");
}
```

**ResponseEntity**

```java
public ResponseEntity<String> responseBodyV2() {
	return new ResponseEntity<>("ok", HttpStatus.OK);
}

public ResponseEntity<HelloData> responseBodyJsonV1() {
	HelloData helloData = new HelloData();
	helloData.setUsername("userA");
	helloData.setAge(20);
	return new ResponseEntity<>(helloData, HttpStatus.OK);
}
```

**@ResponseBody**

```java
@ResponseStatus(HttpStatus.OK)
@ResponseBody
@GetMapping("/response-body-string-v3")
public String responseBodyV3() {
	return "ok";
}
```

@ResponseBody 로 응답을 보낼 때는 Status 설정을 애노테이션으로 해줘야한다. 또한 컨트롤러에 @RestController를 등록하면, 해당 컨트롤러에 모든 메서드에 @ResponseBody가 적용된다.

주의 : @ResponseBody를 빼고, String을 응답하면 해당 경로에 있는 정적리소스를 반환한다.

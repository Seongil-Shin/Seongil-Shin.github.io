---
title: [spring] InvalidDefinitionException
author: 신성일
date: 2022-11-05 18:36:26 +0900
categories: [study, spring]
tags: [spring, exception]
---

# InvalidDefinitionException

@ResquestBody로 받는 매개변수의 클래스에 @NoArgsConstructor를 빼고 넣으니 위와 같은 에러가 발생했다.

```java
@PostMapping("/login")
public String login(@Valid @RequestBody LoginRequestBody loginRequestBody)
```

```java
@Getter
@AllArgsConstructor
@ToString
public class LoginRequestBody {
```

> com.fasterxml.jackson.databind.exc.InvalidDefinitionException: Cannot construct instance of `com.ntscorp.springserver.user.dto.controller.request.LoginRequestBody` (no Creators, like default constructor, exist): cannot deserialize from Object value (no delegate- or property-based Creator)

반면에 `@NoArgsConstructor`를 추가하니 제대로 controller가 실행되었다.

그 이유는 [해당 포스트](https://yeonyeon.tistory.com/221)에서 자세히 설명되어있다

1. spring에서는 Http 메시지 바디를 읽기위해 HttpMessageConverter를 사용한다.
2. `application/json`을 읽기위해 사용되는 컨버터는 `Jackson2HttpMessageConverter`다.
3. `Jackson2HttpMessageConverter`의 read() 에서는 `ObjectMapper`가 사용된다.
4. `ObjectMapper`의 변환과정은 다음과 같다
   1. Object 생성 : 대부분 기본 생성자 이용
   2. Object 필드 인식 : setter 또는 getter 사용
   3. Object 필드에 값 넣기 : Reflection 이용 (setter 이용 x)

위 과정 중 4-1번에서 기본 생성자를 사용하는 과정이 있으므로 기본생성자가 없을 경우 예외가 발생한 것이었다.

물론 [이 포스트](https://juhi.tistory.com/68)에서 설명한 것과 같이 기본 생성자없이도 동작가능하도록 할 수 있으나, 웬만하면 기본 설정을 사용하는 것이 나아보인다.

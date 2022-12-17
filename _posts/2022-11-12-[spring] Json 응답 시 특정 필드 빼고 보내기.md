---
title: json 응답 시 특정 필드 빼고 보내기
author: 신성일
date: 2022-11-12 18:19:26 +0900
categories: [study, spring]
tags: []
---


스프링에서는 Http 메시지 컨버터를 통해 사용자가 보낸 데이터와 서버에서 내보내는 데이터를 자동으로 변환해준다. 만약 다음과 같은 객체를 응답으로 보낼 경우 `Content-Type`이  `application/json`이라면, 스프링은 다음 객체를 자동으로 Json 형식으로 바꾸어 내보낸다.

```java
@AllArgsConstructor
@Getter
public class UserDto {
  private String userId;
  private String password;
}

//
  
UserDto user = new UserDto("userId", "password");
```

```json
{
  "userId":"userid",
  "password":"password"
}
```

그런데 만약 예시에서 `password`는 민감한 데이터이므로 응답으로 보내고 싶지 않다면 어떻게 하면 될까? 응답을 보낼 때 password 필드의 값을 null로 비워서 보낼 수도 있겠지만, 개발자가 실수로 비우지 않을 수도 있고 코드도 깔끔하지 않을 것이다.

## @JsonIgnore

@JsonIgnore은 Jackson 패키지의 애노테이션으로, Json의 직력화 및 역직렬화에서 속성을 무시하는데 사용한다.

위와 같은 상황에서 사용하는 것이 `@JsonIgnore`으로, 클래스 필드에 해당 애노테이션을 추가해주면 HTTP 메시지 컨버터에서 Json으로 직렬화할 때, `@JsonIgnore`가 붙은 필드는 빼고 변환한다.

```java
@AllArgsConstructor
@Getter
public class UserDto {
  private String userId;
  @JsonIgnore
  private String password;
}

//
  
UserDto user = new UserDto("userId", "password");
```

```json
{
  "userId":"userid"
}
```




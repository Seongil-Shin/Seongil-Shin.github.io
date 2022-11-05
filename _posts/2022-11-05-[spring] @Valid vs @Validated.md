---
title: @Valid vs @Validated
author: 신성일
date: 2022-11-05 18:25:26 +0900
categories: [study, spring]
tags: [spring, annotation, valid]
---

## @Valid vs @Validated

**@Valid**

-  JSR-303 표준 객체 제약조건 검증 어노테이션
-  ArgumentResolver에 의해 처리된다.
-  검증에 오류가 있다면 `MethodArgumentNotValidException` 예외가 발생하며 디스패처 서블릿에 기본으로 등록된 예외 리졸버인 `DefaultHandlerExceptionResolver`에 의해 400 에러가 발생한다.
-  스프링에서는 ArgumentResolver에 의해 처리되기에 `@Valid`는 기본적으로 **컨트롤러에서만 동작**하며 다른 계층에서는 검증되지 않는다. 다른 계층에서 파라미터를 검증하기 위해선 **@Validated**와 결합하여야한다.

**@Validated**

-  스프링에서 제공하며, **AOP**기반으로 메소드의 요청을 가로채서 유효성 검증을 해준다.

-  유효성 검증에 실패하면 `ConstraintViolationException` 예외가 발생한다.

-  @Validated를 클래스로 등록하고, @Valid를 파라미터에 등록하면

   -  해당 클래스에 유효성 검증을 위한 인터셉터가 등록된다. 해당 클래스의 메소드들이 호출될 때 AOP 포인트 컷으로써 요청을 가로채서 유효성 검증을 진행한다. 따라서 스프링 빈이라면 컨트롤러가 아니더라도 다른 계층에서도 동작가능하다.

-  클래스레벨이 아닌 메소드 파라미터에 단독으로 등록하는 것 또한 가능하다.

-  @Valid와 다르게 Group Validation까지 가능하도록 구현되었다.

   -  Group Validation이란 객채 유효성 검증에서, 모든 필드를 검사하지 않고 특정 그룹의 필드만 검증하도록 설정하는 것을 얘기한다.

   -  이때 그룹을 지정할 때는 그룹 인터페이스를 정의하고 필드에 그룹을 추가해야한다.

      ```java
      public class AccountLevelManagementCommand {
          @NotBlank(
                  message = "Account ID cannot be empty.",
                  groups = {AccountLevelManagementRequestValidationGroup.class,
                            AccountLevelManagementSubmitValidationGroup.class})
          private String id;

          @NotNull(
                  message = "Level cannot be null.",
                  groups = {AccountLevelManagementSubmitValidationGroup.class})
          private ManagerLevel levelName;

          @NotNull(
                  message = "Operation cannot be null.",
                  groups = {AccountLevelManagementSubmitValidationGroup.class})
          private AccountLevelManagementOperation operation;
      }
      ```

      이 그룹 인터페이스는 비어있어도 좋다.

      ```java
      public interface AccountLevelManagementRequestValidationGroup {
      }
      ```

      상속도 가능하며 이때는 자식 그룹 인터페이스를 등록하면, 부모 그룹까지 모두 등록되게 된다.

      ```java
      public interface SubmitValidation {
      }
      public interface AccountLevelSubmit1 extends SubmitValidation {
      }
      ```

**BindingResult**

@Valid, @Validated를 사용하는 메서드에 파라미터로 주어, 유효성 검사에 실패하면 그에 대한 에러 정보를 포함하고 있는 객체이다.

BindingResult가 없다면 400 오류를 발생하면서 컨트롤러가 호출되지 않지만, 있다면 에러정보를 담고 컨트롤러를 호출한다.

```java
@PostMapping("/article/create")
public String create(@Valid ArticleCreate article, BindingResult bindingResult) {
  if(bindingResult.hasErrors()) {
    return "errorpage";
  }
 // ... 작업
}
```

[참고문서](https://velog.io/@park2348190/Spring%EC%9D%98-Valid-Validated)

[참고문서2](https://sweets1327.tistory.com/54)

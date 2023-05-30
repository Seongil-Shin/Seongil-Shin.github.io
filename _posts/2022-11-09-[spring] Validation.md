---
title: Spring 파라미터 Validation
author: 신성일
date: 2022-11-09 18:19:26 +0900
categories: [study, spring]
tags: []
---

# **[spring] 파라미터 validation**

## **BindingResult**

HTTP 메세지 컨버터에서 발생한 데이터 바인딩 오류를 담아 컨트롤러에서 이용할 수 있는 객체. 다음과 같이 바인딩 오류가 발생할 수 있는 파라미터 바로 다음에 파라미터로서 추가한다.

```java
@PostMapping("/add")
public String addItemV1(@ModelAttribute Item item, BindingResult bindingResult)
```

컨트롤러에서 데이터 바인딩 오류가 발생하면 400 오류를 클라이언트로 전달한다. 하지만 위와 같이 BindingResult를 파라미터에 추가하면, 400 오류를 바로 보내지 않고 컨트롤러를 호출하여 개발자에게 오류를 해결하도록 한다.

```java
@PostMapping("/add")
public String addItemV1(@ModelAttribute Item item, BindingResult bindingResult) {
  // 없으면 바로 400 오류를 보냄
  if (bindingResult.hasErrors()) {
    log.info("errors={}", bindingResult);
    return "validation/v2/addForm";
  }
  ...
}
```

BindingResult 객체는 thymeleaf 등에서 사용할 수 있다. `th:errors="*{itemName}"` 과 같이 접근할 수 있는데, 이를 위해서 개발자가 임의로 BindingResult에 오류를 추가해 줄 수 있다.

```java
// FieldError : 하나의 필드에 오류가 있을 때 추가하는 객체. 스프링에서 바인딩 오류시 넣어줌
bindingResult.addError(new FieldError("item", "itemName", "상품 이름은 필수입니다."));

// ObjectError : 특정 필드가 아닌 여러 필트에 걸쳐 발생하거나, 글로벌 오류일 때 추가
bindingResult.addError(new ObjectError("item", "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice));
```

각각 타임리프에서 접근은 다음과 같이 한다.

```html
<!-- FieldError -->
<div class="field-error" th:errors="*{itemName}">
상품명 오류
</div>

<!-- ObjectError -->
<div th:if="${#fields.hasGlobalErrors()}">
      <p class="field-error" th:each="err : ${#fields.globalErrors()}" th:text="${err}">
        전체 오류 메시지
  		</p> 
</div>
```

<br/>

## **Errors**

BindingResult은 인터페이스이고, Errors 인터페이스를 상속받고 있다. 스프링에서는 `BeanPropertyBindingResult`라는 구현체로 둘 다 구현하여 넘겨주는데, 따라서 BindingResult 대신 Errors 인터페이스를 사용해도 된다.

하지만 Errors는 단순 오류 저장, 조회만을 포함하고 BindingResult는 좀 더 다양한 기능을 제공한다. 또한 관례적으로 BindingResult를 사용하니 BindingResult를 사용하는 것이 좋다.

<br/>

## **Validator**

스프링은 검증을 체계적으로 제공하기 위해 다음 인터페이스를 제공한다

```java
public interface Validator {
 	 	// 이 validator에서 지원하는 클래스인지 확인
    boolean supports(Class<?> clazz);
  	// 검증 로직
    void validate(Object target, Errors errors);
}
```

이 인터페이스를 사용해서 다음과 같이 구현하였다고 해보자.

```java
@Component
public class ItemValidator implements Validator {
	@Override
	public boolean supports(Class<?> clazz) {
		return Item.class.isAssignableFrom(clazz);
	}
  
	@Override
	public void validate(Object target, Errors errors) {
		Item item = (Item) target;
    
		if (item.getPrice() != null && item.getQuantity() != null) {
			int resultPrice = item.getPrice() * item.getQuantity();
			if (resultPrice < 10000) {
				errors.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
      } 
    }
  }
}
```

이렇게 구현한 validator는 스프링의 `WebDataBinder`를 통해 스프링의 도움을 받을 수 있다. 컨트롤러에 다음과 같이 `@InitBinder` 설정을 한 메서드를 추가하면, 해당 컨트롤러에서는 validator를 자동으로 적용할 수 있다.

```java
@InitBinder
public void init(WebDataBinder dataBinder) {
	dataBinder.addValidators(itemValidator);
}
```

이렇게 등록된 검증기를 사용하려면 컨트롤러 메서드 파라미터에 `@Validated` 또는 `@Valid` 애노테이션을 추가하면 된다,

```java
@PostMapping("/add")
public String addItemV6(@Validated @ModelAttribute Item item, BindingResult bindingResult)
```

글로벌로 모든 컨트롤러에 등록할 수도 있다. `@SpringBootApplication` 애노테이션이 붙은 루트 클래스에서 validator를 추가해주면 된다.

```java
@SpringBootApplication
public class ItemServiceApplication implements WebMvcConfigurer {
		public static void main(String[] args) {
			SpringApplication.run(ItemServiceApplication.class, args);
		}
  
  	// 글로벌 적용
		@Override
		public Validator getValidator() {
			return new ItemValidator();
		}
}
```

<br/>

## Bean Validation

애노테이션으로 쉽게 검증 로직을 적용할 수 있는 Bean Validation 2.0(JSR-380) 기술 표준이다. 검증 애노테이션과 여러 인터페이스의 모음이다.

스프링에서 Bean Validation을 사용하기 위해선 `spring-boot-starter-validation`라이브러리를 의존성에 추가해야한다. 추가 후 사용할 땐, 검증하고자하는 필드 위에 검증할 애노테이션을 추가하면 된다.

```java
@Data
public class Item {
	private Long id;
    
	@NotBlank
	private String itemName;
    
	@NotNull
	@Range(min = 1000, max = 1000000)
	private Integer price;
}
```

그리고 컨트롤러에 `@Validated` 또는 `@Valid`를 추가해주면 된다.

```java
@PostMapping("/add")
public String addItem(@Validated @ModelAttribute Item item, BindingResult bindingResult)
```

> javax.validation.constraints.NotNull
>
> org.hibernate.validator.constraints.Range
>
> javax.validation 으로 시작하면 특정 구현에 관계없이 제공되는 표준 인터페이스이고, org.hibernate.validator 로 시작하면 하이버네이트 validator 구현체를 사용할 때만 제공되는 검증 기능이다. 실무에서 대부분 하이버네이트 validator를 사용하므로 자유롭게 사용해도 된다.

스프링은 Bean Validator를 이미 스프링에 완전히 통합해두었다. 스프링부트는 `spring-boot-starter-validation` 라이브러리를 넣으면 자동으로 Bean Validator를 인지하고 스프링에 통합한다. 이때 스프링부트는 자동으로 Bean Validator를  글로벌 Validator로 등록한다. 

이때 개발자가 직접 글로벌 Validator를 등록하면 스프링부트가 추가하는 Bean Validator를 적용되지 않는다. 따라서 Bean Validator를 사용할 때는 직접 등록한 글로벌 Validator를 지워주자.

**검증 순서**

Bean Validator는 바인딩 된 데이터들을 검증한다. 따라서 애초에 바인딩에서 실패한다면 Bean Validator는 실행되지 않고 FieldError를 추가한다.

> 검증 순서
>
> 1. @ModalAttribute 각각의 필드에 타입 변환시도
>    1. 성공하면 다음으로
>    2. 실패하면 typeMismatch로 FieldError 추가
> 2. Validator 적용

**오브젝트 오류**

Bean Validator에서 특정 필드가 아닌 ObjectError를 처리하기 위해선 `@ScriptAssert()`를 사용하면 된다.

```java
@Data
@ScriptAssert(lang = "javascript", script = "_this.price * _this.quantity >= 10000")
public class Item {
	//...
}
```

하지만 이 방법은 제약이 많고 복잡하다. 그리고 실무에서는 검증기능이 해당 객체 범위를 넘어서는 경우도 종종 등장한다(DB 조회 등). 따라서 Bean Validator에서 ObjectError는 @ScriptAssert()를 사용하기 보단 컨트롤러에서 직접 자바코드로 작성하는 것을 권장한다.

**groups**

다음과 같이 검증 애노테이션을 그룹지어 경우에 따라 다른 검증 애노테이션이 동작하도록 설정할 수 있다

```java
@Data
public class Item {
	@NotNull(groups = UpdateCheck.class) 
	private String itemName;
  
	@NotNull(groups = {SaveCheck.class, UpdateCheck.class})
	@Range(min = 1000, max = 1000000, groups = {SaveCheck.class, UpdateCheck.class})
	private Integer price;
}
```

```java
// SaveCheck 그룹의 애노테이션만 validation 적용
@PostMapping("/add")
public String addItemV2(@Validated(SaveCheck.class) @ModelAttribute Item item, BindingResult bindingResult)
```

이때 `@Valid` 애노테이션에는 groups 기능이 없다. 따라서 이때는 `@Validated`를 사용해야한다.

하지만 groups 기능을 사용하면 코드의 복잡도가 증가해서 보통 groups 기능을 사용하기보다는 검증을 달리해야하는 그룹마다 따로 객체를 생성해서 처리하는 편이다.

**@ModelAttribute vs @RequestBody**

Bean Validation은 @RequestBody를 적용하여 파라미터를 받아올 때도 동작한다. 하지만 객체를 바인딩할 떄의 동작은 다소 다르다.

@ModelAttribute에서는 필드 단위로 바인딩이 적용된다. 따라서 한 필드에서 바인딩 실패가 발생했을 경우엔 나머지 필드는 정상 처리하고 validation을 수행한다. 

하지만 @RequestBody은 HttpMessageConver 단계에서 바인딩할 떈 전체 객체 단위로 적용된다. 따라서 한 필드에서 바인딩 오류가 발생한다면 전체 필드가 처리되지 않고 예외가 발생한다. 







## **출처**

[김영한의 스프링 MVC 2편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-2)

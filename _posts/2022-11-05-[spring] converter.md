---
title: converter
author: 신성일
date: 2022-11-05 18:33:26 +0900
categories: [study, spring]
tags: [spring]
---

controller 인자로 enum 받는 문제를 해결했던 것은 enum 타입에 맞춘 `Converter`를 bean에 등록하는 것이었다. 단순히 bean에 등록하는 것으로 controller에서 httpConverter 동작할 때 이것을 적용한 것이다. 이 뒤에 무엇이 있을까 하여 조사해보았다.

스프링에서는 기본적인 타입 변환은 자동으로 지원한다. 따라서 파라미터로 넘어온 문자 데이터가 컨트롤러에서 직접 Integer 타입으로 변환된다.

```java
@GetMapping(path = "/{articleId}")
public String getArticleDetail(@PathVariable long articleId) {
  ...
```

위와 같이 파라미터로 넘어온 문자데이터를 long 타입으로 받아도 스프링에서는 자동으로 변환시켜주기에 문제 없다.

하지만 지원하지 않는 타입도 존재한다. 이럴땐 `Converter 인터페이스`를 사용하면 된다. 이때 주의할 것이 아래 경로의 Converter를 가져와야한다.

> `org.springframework.core.convert.converter.Converter`

Converter는 두 개의 제네릭을 받는다. 첫번째는 입력 타입이고, 두번째는 반환 타입이다.

```java
public class SearchTypeConverter implements Converter<String, SearchType>
```

Converter를 적용하려면 `convert` 메서드를 오버라이드하고, 스프링 빈에 등록해야한다.

```java
@Component
public class SearchTypeConverter implements Converter<String, SearchType> {
	@Override
	public SearchType convert(String value) {
		return SearchType.valueOf(value.toUpperCase());
	}
}
```

이렇게 등록을 하고 나면 스프링에서 해당 입력 타입을 반환타입으로 convert 해야할 때 등록한 메서드가 사용된다.

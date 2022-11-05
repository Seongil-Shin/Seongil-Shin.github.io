---
title: controller에서 enum을 인자로 받기
author: 신성일
date: 2022-11-05 18:32:26 +0900
categories: [study, spring]
tags: [spring, enum, controller]
---

## controller 인자로 enum 받기

검색 API를 개발하는 과정에서 다음과 같이 검색 타입을 enum 으로 지정하였고, controller의 param으로 받으려고 했다.

```java
@Getter
@AllArgsConstructor
public enum SearchType {
	TITLE("title"),
	CONTENT("content"),
	HASHTAG("hashtag"),
	NICKNAME("nickname");

	private final String type;
}
```

```java
public class ArticleSearchQuery {
	public Optional<Integer> offset;
	public Optional<Integer> limit;
	public Optional<Integer> category;
	@NotBlank
	public String keyword;
	@NotNull
	public SearchType type;
}
```

```java
@GetMapping(path = "/search")
public ResponseEntity<ResponseWrapper<ArticleListDto>> searchArticles(
	@Valid ArticleSearchQuery articleSearchQuery) {
   ...
}
```

하지만 SearchType을 요청을 ModelAttribute로 변환하는 과정에서 다음과 같은 오류가 났다.

```text
Resolved [org.springframework.web.method.annotation.ModelAttributeMethodProcessor$1: org.springframework.validation.BeanPropertyBindingResult: 1 errors<EOL>Field error in object 'articleSearchQuery' on field 'type': rejected value [hashtag];
```

[Spring's @RequestParam with Enum](https://stackoverflow.com/questions/39774427/springs-requestparam-with-enum) 질문의 답변을 보면 Converter를 구현하고 그것을 빈에 등록하면, 스프링 부트가 자동으로 컨버터를 로드하여 데이터를 변환할 때 쓴다고 한다.

```java
@Component
public class SearchTypeConverter implements Converter<String, SearchType> {
	@Override
	public SearchType convert(String value) {
		return SearchType.valueOf(value.toUpperCase());
	}
}
```

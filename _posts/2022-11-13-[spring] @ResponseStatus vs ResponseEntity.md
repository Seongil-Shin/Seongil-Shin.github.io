---
title: ResponseStatus vs ResponseEntity
author: 신성일
date: 2022-11-13 18:23:26 +0900
categories: [study, spring]
tags: []
---

​	

## @ResponseStatus

응답으로 보낼 데이터의 HttpStatus를 명시해주는 방법이다. 컨트롤러에서 Body 데이터만 반환하는 경우 HttpStatus를 명시하기 위해 사용한다.

```java
@ResponseStatus(HttpStatus.OK)
@GetMapping("/article")
public ArticleListDto getArticles(SelectArticlesQuery query) {
		ArticleListDto articleListDto = articleService.getArticleList(query);
		return articleListDto;
}
```

<Br/>

## ResponseEntity

스프링에선 HTTP 요청 혹은 응답을 나타내기 위해 제공하는 HttpEntity라는 클래스가 존재하며, HttpEntity는 HttpHeader와 HttpBody를 포함하는 클래스이다.

이러한 HttpEntity를 상속하여 추가적으로 HttpStatus 속성을 더 가지는 클래스가 RequestEntity, ResponseEntity 클래스이다. 

제네릭으로 선언되어있으며 해당 부분은 HttpBody 타입을 나타낸다.

```java
@GetMapping("/article")
public ResponseEntity<ArticleListDto> getArticles(SelectArticlesQuery query) {
		ArticleListDto articleListDto = articleService.getArticleList(query);
		return ResponseEntity.ok(response);
}
```

<br/>

## @ResponseStatus의 단점

@ResponseStatus는 단 하나의 응답 상태 코드를 지정하는 것 외에 다른 작업이 불가능하다는 것이 가장 큰 단점이다.

ResponseEntity의 경우 필요하다면, 컨트롤러 메서드 내부에서 오류 상태 코드를 지정하여 오류를 보낼 수 있다. 또한 Header에 대한 설정도 가능하다. 따라서 @ResponseStatus에 비해 유연성이 좋다.

<br/>

## 출처

https://joojimin.tistory.com/54


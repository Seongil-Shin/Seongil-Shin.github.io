---
title: [spring] @ModelAttribute와 @NoArgsConstructor 동시 사용 이슈
author: 신성일
date: 2022-11-05 18:15:26 +0900
categories: [study, spring]
tags: [spring, binding]
---

# @ModelAttribute와 @NoArgsConstructor 추가 시 아무것도 안들어가는 이슈

제목 그대로 다음과 같이 `@NoArgsConstructor` 애노테이션을 `@ModelAttribute`로 적용되는 컨트롤러 파라미터 클래스에 추가하니 제대로 쿼리를 날려도 아무것도 안들어가는 이슈가 발생했다.

```java
@GetMapping("/article")
public ResponseEntity getArticles(@ModelAttribute SelectArticlesQuery query)
```

```java
@Getter
@AllArgsConstructor
@NoArgsConstructor
@ToString
public class SelectArticlesQuery {
	private Integer offset;
	private Integer limit;
	private Integer category;
}
```

```test
요청 : /article?limit=20&offset=10
toString() => SelectArticlesQuery(offset=null, limit=null, category=null)
```

<br/>

이 이슈에 대한 해답은 [이 포스트](https://steady-coding.tistory.com/489)에서 찾을 수 있었다.

Spring은 URL 파라미터 또는 POST Form Data 형태의 파라미터를 위와 같은 특정 클래스로 자동으로 바인딩해주는데 이때 반환되는 객체를 커맨드 객체라고 부른다. 그리고 커맨드 객체의 매핑규칙은 다음과 같다.

1. NoArgsConstructor와 AllArgsConstructor 둘 다 있는 경우
   -  NoArgsConstructor를 호출하고, setter 호출하여 param을 필드에 각각 초기화 한다.
2. AllArgsConstructor만 있는 경우
   -  AllArgsConstructor을 호출하여 param을 필드에 각각 초기화 한 뒤, setter 호출하여 param을 필드에 각각 다시 초기화하여 덮어씌운다.

따라서 만약 @NoArgsConstuctor가 있다면 @Setter를 통해 setter 메서드를 선언해주어야 필드를 초기화 할 수 있게 된다. 따라서 위 예시에서 `SelectArticlesQuery`을 다음과 같이 변경하면 문제는 해결된다.

```java
@Getter
@Setter // 추가
@AllArgsConstructor
@NoArgsConstructor
@ToString
public class SelectArticlesQuery {
	private Integer offset;
	private Integer limit;
	private Integer category;
}
```

```text
요청 : /article?limit=20&offset=10
toString() => SelectArticlesQuery(offset=10, limit=20, category=null)
```

## 참고

-  https://steady-coding.tistory.com/489

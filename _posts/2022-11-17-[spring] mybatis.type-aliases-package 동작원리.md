---
title: mybatis.type-aliases-package
author: 신성일
date: 2022-11-17 16:19:26 +0900
categories: [study, spring]
tags: []
---

mybatis mapper에서 requestType이나 parameterType으로 클래스를 참조할 때 다음과 같이 패키지명을 다 써줘야하는 불편함이 있다.

```xml
<mapper namespace="com.hello.spring.article.mapper.ArticleMapper">
    <select id="selectArticles" resultType="com.hello.spring.article.dto.ArticleDto">
      ...
```

이럴 땐 `application.properties`에서 다음과 같이 설정해주면 쿼리 태그에서는 패키지명을 써주지 않고 클래스명만 써줘도 된다.

```properties
mybatis.type-aliases-package=com.hello.spring
```

```xml
<mapper namespace="com.hello.spring.article.mapper.ArticleMapper">
    <select id="selectArticles" resultType="ArticleDto">
      ...
```

스프링에서 `mybatis.type-aliases-package`에 설정된 패키지를 보고 그 하위 경로에서 클래스를 찾는 방식이다.

<br/>

### 주의점

`mybatis.type-aliases-package`에서 설정한 패키지 하위 경로를 모두 살펴보는 방식으로 동작하므로, 같은 클래스가 두 개 이상 존재하면 에러가 난다.

```
// 파일구조
dto
 > test
   > ArticleDto
 > ArticleDto
```

```
Caused by: org.apache.ibatis.type.TypeException: The alias 'ArticleDto' is already mapped to the value 'com.hello.spring.article.dto.ArticleDto'.
```

이럴 땐 아래와 같이 둘 중 하나를 `@Alias` 애노테이션으로 다른 이름으로 지정해주면 해결된다.

```java
@Alias("ArticleDto2")
public class ArticleDto
```

<br/>

## 참고

- https://yjh5369.tistory.com/entry/Spring-boot-mybatis-alias-%EC%82%AC%EC%9A%A9%EB%B2%95

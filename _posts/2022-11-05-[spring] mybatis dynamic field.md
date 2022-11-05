---
title: [spring] mybatis dynamic field
author: 신성일
date: 2022-11-05 18:19:26 +0900
categories: [study, spring]
tags: [spring, mybatis]
---

## mybatis dynamic field

게시글 검색을 개발하는 도중 검색의 대상이 되는 field(title | nickname | content)를 동적으로 mybatis 쿼리문 상에 사용해야할 일이 있었다. 따라서 다음과 같이 `#{}` 를 사용하여 변수를 넣었으나 오류가 발생했다.

```sql
<where>
	<when test="type != 'hashtag'">
		atc.#{type} LIKE concat('%', #{keyword}, '%')
	</when>
</where>
```

[스택오버플로우 질문의 답변](https://stackoverflow.com/questions/25877458/how-to-pass-dynamic-fields-and-values-using-mybatis-3)에서 `#{}`은 mybatis에서 자동으로 값을 `''`로 감싸기에 그냥 replace하는 `${}`라는 답변을 얻었고 해결할 수 있었다.
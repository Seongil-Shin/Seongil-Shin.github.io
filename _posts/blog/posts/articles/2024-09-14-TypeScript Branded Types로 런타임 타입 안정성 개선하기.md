---
title: TypeScript Branded Types로 런타임 타입 안정성 개선하기
author: 신성일
date: 2024-09-14 19:00:00 +0900
categories: study, typescript
tags:
  - "#study"
---
https://siosio3103.medium.com/typescript-branded-types%EB%A1%9C-%EB%9F%B0%ED%83%80%EC%9E%84-%ED%83%80%EC%9E%85-%EC%95%88%EC%A0%95%EC%84%B1-%EA%B0%9C%EC%84%A0%ED%95%98%EA%B8%B0-768222c8df0d


## 문제

만약 타입이 아래처럼 정의되어있다고 해보자

```ts
type User = {  
	id: string  
	name: string  
}  
  
type Post = {  
	id: string  
	ownerId: string;  
	comments: Comments[]  
}  
  
type Comments = {  
	id: string  
	timestamp: string  
	body: string  
	authorId: string  
}
```

하나의 Post에는 어떤 User인지랑 여러 Comment를 가지고 있다. 그런데 id들이 전부 string 타입이기에 만약 다음과 같이 코드를 짜면 타입스크립트는 정상으로 처리한다.

```ts
async function getCommentsForPost(postId: string, authorId: string) {  
	const response = await api.get(`/author/${authorId}/posts/${postId}/comments`)  
	return response.data  
}  
  
const comments = await getCommentsForPost(user.id, post.id) // 순서가 반대
```

런타임에서는 물론 다르게 동작한다. 이러한 경우 `Branded Types`가 사용될 수 있다.

## Branded Types

Branded Type은 기본 타입과 brand 프로퍼티에 있는 객체 리터럴을 결합하여 타입에 추가할 수 있다.

```ts
type Brand<K, T> = K & { __brand: T }  
type UserID = Brand<string, "UserId">  
type PostID = Brand<string, "PostId">

type User = {  
	id: UserID;  
	name: string  
}  
type Post = {  
	id: PostID;  
	ownerId: string;  
	comments: Comments[]  
}  
type Comments = {  
	id: CommentID  
	timestamp: string  
	body: string  
	authorId: UserID  
}

async function getCommentsForPost(postId: PostID, authorId: UserID) {  
	const response = await api.get(`/author/${authorId}/posts/${postId}/comments`)  
	return response.data  
}  
const comments = await getCommentsForPost(user.id,post.id) // TS 오류 발생
```

이제 타입스크립트에서 오류를 발견할 수 있지만, 몇가지 단점이 있다
- 타입에 태그를 지정하는데 사용되는 `__brand` 프로퍼티는 빌드 시점에만 존재하는 프로퍼티다
- `__brand` 프로퍼티는 런타임에서는 표시되지 않기에 개발자가 이를 사용하려고하면 문제가 되지만 자동완성에는 보인다.
- `__brand` 프로퍼티에 대한 제약이 없기에 branded types를 복제할 수 있다.

## 더 나아진 Branded Type

- `__brand`라는 하드코딩 프로퍼티 대신 계산된 프로퍼티키를 사용할 수 있다
- unique symbol을 사용해 이미 사용된 key의 중복을 막을 수 있다
- Brand 유틸리티를 자체 파일에 작성하여 해당 프로퍼티의 읽기 접근을 차단할 수 있다.

```ts
declare const __brand: unique symbol  
type Brand = { [__brand]: B }  
export type Branded<T, B> = T & Brand
```

## Branded Types 의 유용성

- 명확성 : 변수의 용도를 명확하게 표현할 수 있다.
- 안정성과 정확성 : 코드를 더 쉽게 추론하고 타입 불일치나 호환되지 않는 타입에 관한 에러를 잡아 이슈를 막을 수 있다
- 유지보수성 : 모호함을 줄여 코드를 읽기 쉽게 만들어준다. 


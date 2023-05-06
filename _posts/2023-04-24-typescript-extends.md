---
title: typescript 조건부타입(extends)
author: 신성일
date: 2023-04-24  22:30:00 +0900
categories: [study, typescript]
tags: []
---

타입 스크립트 2.8부터 다음과 같은 `조건부 타입`을 사용할 수 있다.

```typescript
T extends U ? X : Y
```

 뜻은 T가 U의 서브타입이거나 같은 타입이면 X, 아니면 Y 타입을 할당한다는 것이다. 

## T가 유니온 타입일 경우

다음과 같이 T가 유니온 타입일 경우가 있다.

```typescript
type T = "A" | "B" | "C"
```

이러한 경우엔 분배법칙이 성립된다.

```typescript
{"A" extends U ? X : Y} | {"B" extends U ? X : Y} | {"C" extends U ? X : Y}
```

<br/>

## 출처

https://mugglim.tistory.com/13
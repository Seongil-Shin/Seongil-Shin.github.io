---
title: Graph ql back
author: 신성일
date: 2021-06-19 12:35:44 +0900
categories: [study, back-end]
tags: [GraphQL]
---

## Qraph QL 이 해결할 수 있는 문제

- over-fetching : 요청한 것보다 많이 보내줌. ex) username만 필요한데, 프로필 사진도 보냄
- under-fetching : 하나의 작업을 위해 여러 요청을 보내게 하는 것.

- QraphQL에는 URL이 존재하지 않음.
  - 하나의 엔드포인트만 있고, 그곳으로 쿼리를 보내면, 쿼리에 맞춰서 알아서 데이터를 보내준다.
  - 즉, 정확히 요청한 데이터만 보내준다.

## 시작

https://www.npmjs.com/package/graphql-yoga

- yarn add graphql-yoga

  - 시작할때, 타입스크립트로 시작을 하든, 바벨 설정을 해줘야한다.

- 기본 구조

  ```js
  import { GraphQLServer } from "graphql-yoga";

  const server = new GraphQLServer({});

  server.start(() => console.log("graph Ql server running"));
  ```

  - 위 코드는 스키마가 정의되어있지 않기에 오류가 발생함.

## Scheme

- 무엇을 받고, 무엇을 줄지에 대한 설명.
- Query : 정보를 받을 때 사용
- Mutation : 데이터베이스 등에서 데이터를 바꿀때 사용.

## 구조

```js
// /graphql/resolvers.js
const resolvers = {
   Query: {
      name: () => "seongil",
   },
};
export default resolvers;


// /graphql/schema.graphql
type Query {
   name: String!
}

// index.js
...
import resolvers from "./graphql/resolvers";

const server = new GraphQLServer({
   typeDefs: "graphql/schema.graphql",
   resolvers: resolvers,
});
...
```

위와 같이 resolver와, Query를 선언해주고, index.js에서 server와 연동시켜주면 된다.

위와 같이 할 경우, 아래와 같이 Query를 보내면

```js
Query {
    name
}
```

아래와 같이 오게 된다.

```js
{
  "data": {
    "name": "seongil"
  }
}
```

## 구조 : 좀 더 자세히

schema를 다음과 같이 했다.

```js
type Query {
   name: String!
}
```

이 의미는 name에 String 데이터를 반환하는데, !를 주어 required라고 표시하는 것이다.

resolver는 데이터를 어떻게 보내줄 것인지 자바스크립트로 작성하는 것이다.

그럼 객체를 보내고 싶을때는 어떻게 할까

```js
// /graphql/resolvers.js
const seongil = {
   name: "seongil",
   age: 24,
   gender: "male",
};
const resolvers = {
   Query: {
      name: () => seongil,
   },
};
export default resolvers;


// /graphql/schema.graphql
type Person {
   name: String!
   age: Int!
   gender: String!
}

type Query {
   person: Person!
}
```

이렇게 해주면 된다.

요청은 다음과 같이 보낸다. 객체 안의 요소를 하나하나 선택하여 받아올 수 있다.

```js
Query {
    person {
        name
        age
    }
}
```

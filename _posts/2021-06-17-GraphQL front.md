---
title: Graph ql front
author: 신성일
date: 2021-06-19 12:35:44 +0900
categories: [study, graphql]
tags: [GraphQL]
---

## 설치

yarn add @apollo/react-hooks apollo-boost graphql

## 사용법

- GraphQL API로 요청을 보낼때는, 요청문을 axios나 fetch를 사용하여 POST 메소드로 보내야함.

- 하지만 Apollo를 사용하면, 위와같이 하지 않아도 된다.
  - 먼저, Apollo client를 만들어야함.

## Apollo client 셋업

다음과 같이, apollo.js 파일을 생성하고, client를 만들어주자. uri에는 graphQL 서버의 주소가 들어가면 된다.

```js
// src/apollo.js
import ApolloClient from "apollo-boost";

const client = new ApolloClient({
  uri: "http://localhost:4000/",
});

export default client;
```

그리고 다음과 같이 index.js에서 연결하자

```js
// src/index.js
...
import client from "./apollo";
import { ApolloProvider } from "@apollo/react-hooks";

ReactDOM.render(
   <ApolloProvider client={client}>
      <App />
   </ApolloProvider>,
   document.getElementById("root")
);
```

## 쿼리 작성 및 사용법

```js
import { useQuery } from "@apollo/client";
import { gql } from "apollo-boost";
```

아래와 같이 쿼리문 작성

```js
const GET_MOVIES = gql`
  {
    movies {
      id
      medium_cover_image
    }
  }
`;
```

아래와 같이 사용. useQuery에 쿼리문을 넣고, loading, error, data를 얻을 수 있다.

```js
const { loading, error, data } = useQuery(GET_MOVIES);

if (loading) {
  return "loading...";
} else if (data && data.movies) {
  return data.movies.map((m) => <h1>{m.id}</h1>);
}
```

- paremeter가 있는 get

  아래와 같이 쿼리문 작성. paremeter가 없을 때는 그냥해도 되지만, 있을 때는 다음과 같이 query getMovie($id:Int!) 라는 이름으로 불러주자.

  이때, 꼭 graphQL 서버에 있는 이름으로 호출할 필요는 없다. 즉, 다음코드에서 getMovie 대신 get만 써도 된다는 것이다.

  타입만 잘 적어주자.

  ```js
  const GET_MOVIE = gql`
    query getMovie($id: Int!) {
      movie(id: $id) {
        id
        title
        medium_cover_image
        description_intro
      }
    }
  `;
  ```

  호출 : 나머지는 똑같되, useQuery의 두번째 인자에 객체를 넣고, variable 필드에 파라미터를 넣어주자

  ```js
  const Detail = () => {
    const { id } = useParams();
    const { loading, data } = useQuery(GET_MOVIE, {
      variables: { id: parseInt(id) },
    });
  
    if (loading) {
      return "loading";
    }
    if (data && data.movie) {
      return data.movie.title;
    }
  };
  ```

## Apollo Cache

- react apollo가 뭔가를 얻으면 그것을 저장하여 같은 데이터를 요청하면, 그대로 캐시에 있는 것을 보여준다.

- 모든 걸 apollo client에서 자동으로 해준다.

## Local State

- Apollo cache에 있는 데이터를 client 에서 변경하여 사용하기

- client에서 만든 데이터 가져오기

  ```js
  // src/routes/Home.js
  const GET_MOVIES = gql`
    {
      movies {
        id
        medium_cover_image
        isLiked @client
      }
    }
  `;
  ```

  - 이렇게 @client를 붙여줘야 apollo가 백엔드로 보내지 않고 client의 데이터임을 안다.

- 데이터 넣기

  - 아래와 같이, resolvers 안에 넣고자하는 데이터 타입에 넣고자하는 필드를 추가하면 된다.

  ```js
   resolvers: {
        Movie: {
           isLiked: () => false, // 디폴트로 false 값 지니도록.
        },
     },
  ```

- 데이터 수정하기

  - 아래와 같이, resolver 안에 Mutation 을 추가하고, 쿼리에서와 똑같이 작성하면 된다.

  ```js
   resolvers: {
       ...
        Mutation: {
           likeMovie: (_, { id }, { cache }) => {
              cache.modify({
                 id: `Movie:${id}`,
                 fields: {
                    isLiked: (isLiked) => !isLiked,
                 },
              });
           },
        },
     },
  ```

  - 이때, cache.modify를 통해 변경한다.
  - cache.modify
    - id : 어느 데이터를 변경할 것인지 명시. apollo dev tool로 형식을 확인할 수 있다.
      - 보통 데이터 타입 + id 로 구성
    - fields : 변경하고자하는 필드와 변경 방식을 정의한다.

- 데이터 변경 요청

  - 아래와 같이 @client만 붙이고 똑같이 하면 된다.
  - useMutation을 사용한다.
  - 이 hook으로 만들어진 함수를 호출하면, 요청이 날아간다.

  ```js
  const LIKE_MOVIE = gql`
     mutation likeMovie($id: Int!) {
        likeMovie(id: $id) @client
     }
  `;
  ...
  const [likeMovie] = useMutation(LIKE_MOVIE, {
        variables: { id: parseInt(id) },
     });
  ...
  <button onClick={likeMovie}>{isLiked ? "Unlike" : "Like"}</button>
  ```

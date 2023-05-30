---
title: next.js - data fetching
author: 신성일
date: 2023-01-23 22:00:00 +0900
categories: [study, next.js]
tags: [next.js]
---

next.js에서 사용자의 요청을 받고, 페이지를 만들어낼 때 외부 서버로 필요한 데이터를 요청해야할 때 사용하는 기법이다. `data fetching`은 **기본적으로 페이지 컴포넌트에서 사용할 수 있고** 경우에 따라 Custom App 컴포넌트에서도 사용할 수 있다.

next 13부터는 `app/` 구조를 사용한다면 일반 컴포넌트에서도 가능하다고 하지만, 현재(2023.01)는 next 13의 `app/` 구조가 베타버전이라 `pages/` 구조를 기준으로 작성하였다.

> **Note**: Next.js 13 introduces the `app/` directory (beta). This new directory has support for [colocated data fetching](https://beta.nextjs.org/docs/data-fetching/fundamentals) at the component level, using the new React `use` hook and an extended `fetch` Web API.
>
> [Learn more about incrementally adopting `app/`](https://beta.nextjs.org/docs/upgrade-guide).

<br/>

## getStaticProps

- 블로그 게시글처럼 빌드 시 이외에 변경되지 않는 값을 불러오고 싶다면 getStaticProps을 사용한다.

```js
export async function getStaticProps(context) {
  return {
    props: {}, // 페이지 컴포넌트에서 필요한 props 값 주입
  };
}
```

- `getStaticProps`가 만드는 건 **HTML, JSON 파일**이다.
  - HTML에는 렌더링된 컴포넌트들이 포함되고, JSON 파일은 `getStaticProps`의 결과가 저장된다.
- 성능이 좋아진다.
  - 빌드 시 한번 호출되어 static 페이지를 생성하고, 사용자 요청이 이 static 파일을 반환하기에 별도의 계산이 필요없다.
  - `getStaticProps`는 HTML, JSON 파일을 만드므로 CDN에 캐시된다.

### 실행시점

`getStaticProps`는 항상 server-side에서 실행된다. 따라서 DB 접근과 같은 민감한 작업을 수행할 수 있다. 실행 시점은 다음과 같다.

- `next build`시에 항상 실행된다.
- `fallback: true`일 때는 백그라운드에서 실행된다.
- `fallback: blocking` 일 때는 최초 페이지 접근 시에 실행된다.
- `revalidate`를 사용하면 백그라운드에서 실행된다.
- `revalidate()`를 사용하면 최초 페이지 접근 시 백그라운드에서 실행된다.

### 제약사항

- `getStaticProps`는 static HTML을 만드므로 유저 `request`에 접근할 순 없다. 만약 접근해야한다면 [Middleware](https://nextjs.org/docs/advanced-features/middleware)를 추가하면 된다.
- **페이지 컴포넌트에서만** 사용할 수 있다. `_app`, `_document`, `_error`에서도 사용할 수 없다
- production에서는 빌드타임에 한번 실행되지만, 개발 시에는 매 요청마다 실행된다.

### Preview mode

[preview mode](https://nextjs.org/docs/advanced-features/preview-mode)를 사용해 빌드타임 대신에 **request time**에 실행시킬 수 있다.

<Br/>

## **getStaticPaths**

- [Dynamic Route](https://nextjs.org/docs/routing/dynamic-routes)를 사용하는 페이지에서, 빌드 시 정적으로 렌더링할 경로 설정.
- `getStaticPaths`에서 반환된 경로는 모두 정적으로 빌드타임에 렌더링되어 정적으로 저장된다.
- 이곳에 정의되지 않은 하위 경로는 접근해도 페이지가 안뜬다.

```js
// pages/posts/[id].js

export async function getStaticPaths() {
  return {
    paths: [{ params: { id: '1' } }, { params: { id: '2' } }],
    fallback: false, // false, true, 'blocking'
  }
}

export async function getStaticProps(context) {
  return
    props: { post: {} },
  }
}

export default function Post({ post }) {
  // Render post...
}
```

- `getStaticPaths`는 `getStaticProps`와 함께 사용해야한다.
  - `getStaticPaths`에서 리턴한 `paths`마다 `getStaticProps`가 실행됨
  - `getStaticPaths`에서 `fallback:true`를 반환하면 `getStaticProps`는 백그라운드에서 실행된다.
  - `getStaticPaths`에서 `fallback:blocking`을 반환하면 `getStaticProps`는 최초 호출 시 실행된다.

### 제약사항

- `getStaticPaths`는 `getServerSideProps`와 함께 쓰일 수 없음.
- production에서는 빌드타임에 한번 실행되지만, 개발 시에는 매 요청마다 실행된다.

### 수요에 따라 paths 생성 (Generating paths on-demand)

- `paths`로 반환한 페이지가 너무 많으면 빌드시 시간이 오래 걸릴 수 있다. 따라서 빈 배열을 반환하고, 페이지에 최초 접근 시에 그 페이지를 렌더하도록 `fallback: blocking`과 함께 쓰일 수 있다.

```js
export async function getStaticPaths() {
  // 페이지 최초 접근시에만 렌더
  if (process.env.SKIP_BUILD_STATIC_GENERATION) {
    return {
      paths: [],
      fallback: "blocking",
    };
  }

  // SKIP_BUILD_STATIC_GENERATION=false 일땐 paths 생성.
  const res = await fetch("https://.../posts");
  const posts = await res.json();

  const paths = posts.map((post) => ({
    params: { id: post.id },
  }));

  return { paths, fallback: false };
}
```

<Br/>

## Incremental Static Regeneration

static 페이지를 다시 빌드할 필요가 있을 때, 전체 페이지를 다시 빌드할 필요없이 각 페이지를 일정시간마다 자동으로 빌드해서 새롭게 업데이트 된 페이지를 보여주는 방법이다.

사용법은 `getStaticProps`의 반환 객체에 `revalidate`를 추가하면 된다.

```js
export async function getStaticProps() {
  const res = await fetch("https://.../posts");
  const posts = await res.json();

  return {
    props: {
      posts,
    },
    // Next.js will attempt to re-generate the page:
    // - When a request comes in
    // - At most once every 10 seconds
    revalidate: 10, // In seconds
  };
}
```

1. 첫 요청이 오면 빌드 시 만든 static 페이지를 반환한다.
2. 10초가 지난 후 첫 요청에는 빌드 시 만든 static 페이지를 반환한다.
3. next.js가 해당 페이지를 백그라운드에서 regenerate한다.
4. 페이지가 정상적으로 빌드되면 next.js는 이전 페이지를 invalidate 시키고, 새로운 페이지를 보여준다. 페이지 빌드가 실패하면 이전 페이지를 반환하고, 새로운 요청이 들어올 시 다시 빌드를 진행한다.

### on-demand Revalidation

특정 주기마다 regenerate하는 대신, 필요할 때마다 업데이트하고 싶을 떄 사용한다.

1. secret key를 생성후, 다음 주소로 요청을 보낸다.

   ```
   https://<your-site.com>/api/revalidate?secret=<token>
   ```

2. secret key를 환경변수에 추가하고, `api/revalidate`를 다음과 같이 구성한다.

   ```js
   // pages/api/revalidate.js

   export default async function handler(req, res) {
     if (req.query.secret !== process.env.MY_SECRET_TOKEN) {
       return res.status(401).json({ message: 'Invalid token' })
     }

     try {
       await res.revalidate('/path-to-revalidate')
       return res.json({ revalidated: true })
     } catch (err)
       return res.status(500).send('Error revalidating')
     }
   }
   ```

### 주의사항

- CDN에서 캐시가 일어나면 ISR이 적절하게 동작하지 않는다. `useCdn : false` 옵션을 사용할 수 있다.

<br/>

## getServerSideProps

- 매 요청마다 새롭게 값을 불러와야할 경우 사용.

  ```js
  export async function getServerSideProps(context) {
    return {
      props: {}, // will be passed to the page component as props
    };
  }
  ```

- 모든 요청에 대한 계산을 해야하고, CDN 캐시 적용이 되지 않기에 더 느리다. 따라서 꼭 매번 새롭게 불러와야할 경우에만 사용한다.

  - 또한 꼭 첫 html에 데이터가 반영되지 않아도 되는 페이지라면, 클라이언트 단에서 데이터를 가져오는 것이 좋다. (next에서 제작한 [swr](https://nextjs.org/docs/basic-features/data-fetching/client-side#client-side-data-fetching-with-swr)을 사용하면 편하다)

- `getServerSideProps`는 **빌드시 번들로 묶이지 않는다**. 따라서 클라이언트 사이드에 전달도 되지 않는다. 그렇기에 서버사이드에서만 실행해야하는 코드(DB 호출)를 사용하기에 좋다.

  - 브라우저에서 실행되지 않기에 `window.document`와 같은 브라우저 API를 사용할 수 없다.

### Context

`context`를 인자로 받는데, 다음과 같은 필드가 포함되어있다.

- params : [dynamic route](https://nextjs.org/docs/routing/dynamic-routes)를 사용 시, 라우팅 파라미터가 담겨있다.
- req : [HTTP IncomingMessage object](https://nodejs.org/api/http.html#http_class_http_incomingmessage)에 `cookies`를 포함한 객체이다.
- res : [HTTP response object](https://nodejs.org/api/http.html#http_class_http_serverresponse) 객체
- query : dynamic route 파라미터를 포함한 쿼리스트링이 담겨있다.
- preview : 페이지가 [Preview Mode](https://nextjs.org/docs/advanced-features/preview-mode)라면 true이고, 아니면 false이다
- previewData : `setPreviewData`로 설정된 preview 데이터가 담겨있다.
- resolveUrl : URL의 정규화된 정보를 포함함
- locale : 설정된 locale 값
- locales : 지원하는 locales 포함
- defaultLocale : default locale로 설정된 값

### Return values

getServerSideProps에서는 다음과 같은 데이터를 반환할 수 있고, 각 반환값에 따라 페이지 렌더링이 달라진다.

- props : 페이지 컴포넌트로 전달되는 데이터를 담는다. 일반적으로 사용하는 반환값으로, props로 넘어간 데이터 가지고 페이지 컴포넌트가 렌더링한다.

  ```js
  export async function getServerSideProps(context) {
    return {
      props: { message: `Next.js is awesome` }, // will be passed to the page component as props
    };
  }
  ```

- notFound : boolean 값을 담는다. `true`일 경우 페이지 컴포넌트를 렌더링하지 않고 404 코드를 낸다.

  ```js
  export async function getServerSideProps(context) {
    const res = await fetch(`https://.../data`);
    const data = await res.json();

    if (!data) {
      return {
        notFound: true,
      };
    }

    return {
      props: { data }, // will be passed to the page component as props
    };
  }
  ```

- redirect : 다른 페이지로 리다이렉트 시킬 때 사용한다. `statusCode` 필드를 추가할 수 있으나, 그럴려면 `permanent` 필드를 설정하면 안된다. ([redirect permanent란?](https://forlater.tistory.com/371))

  ```js
  export async function getServerSideProps(context) {
    const res = await fetch(`https://.../data`);
    const data = await res.json();

    if (!data) {
      return {
        redirect: {
          destination: "/",
          permanent: false,
        },
      };
    }

    return {
      props: {}, // will be passed to the page component as props
    };
  }
  ```

### with Typescript

타입스크립트와 함께 사용할 때 타이핑 기법들은 다음과 같다.

```typescript
import { GetServerSideProps } from 'next'

type Data = { ... }

export const getServerSideProps: GetServerSideProps<{ data: Data }> = async (context) => {
  const res = await fetch('https://.../data')
  const data: Data = await res.json()

  return {
    props: {
      data,
    },
  }
}
```

추론된 타입을 사용하고 싶으면 다음과 같이 할 수 있다.

```typescript
import { InferGetServerSidePropsType } from 'next'

type Data = { ... }

export const getServerSideProps = async () => {
  const res = await fetch('https://.../data')
  const data: Data = await res.json()

  return {
    props: {
      data,
    },
  }
}

function Page({ data }: InferGetServerSidePropsType<typeof getServerSideProps>) {
  // will resolve data to type Data
}

export default Page
```

<br/>

## getInitialProps

`getServerSideProps`와 같이 서버사이드에서 페이지 컴포넌트를 렌더하기 전에 필요한 데이터를 불러오기 위한 메서드이다( `getStaticProps`, `getStaticPaths`의 기능은 없음). next v9 이전에 사용하였으며 그 이후에는 `getServerSideProps`가 사용된다. 하지만 레거시 코드를 읽을 때 필요하니 알아둘 필요가 있다.

```js
export default function Page({ stars }) {
  return <div>Next stars: {stars}</div>;
}

Page.getInitialProps = async (ctx: NextPageContext) => {
  const res = await fetch("https://api.github.com/repos/vercel/next.js");
  const json = await res.json();
  return { stars: json.stargazers_count };
};
```

### getServerSideProps와 차이점

- **빌드시 번들로 묶인다.** 첫 페이지 렌더시에는 서버에서 실행되는데, `next/link`, `next/router`로 페이지 네비케이팅 됐을 시에는 클라이언트에서 실행된다.
  - but `getInitialProps`가 Custom App 컴포넌트에 있고, 네비게이팅된 페이지가 `getServerSideProps`를 사용중이라면, 서버에서 실행된다.
- 인자로 받는 `context`의 구성이 다르다.
  - pathname: 현재 라우트.
  - req : [HTTP IncomingMessage object](https://nodejs.org/api/http.html#http_class_http_incomingmessage)에 `cookies`를 포함한 객체이다.
  - res : [HTTP response object](https://nodejs.org/api/http.html#http_class_http_serverresponse) 객체
  - query : dynamic route 파라미터를 포함한 쿼리스트링이 담겨있다.
  - asPath : 브라우저에서 보여지는 쿼리를 포함한 실제 path
  - err : 렌더링 중 일어난 에러

### 주의사항

- 한 페이지 컴포넌트에서는 하나의 `getInitialProps`만 실행됨.

  - 따라서 `_app.tsx`와 페이지 컴포넌트 둘 다 `getInitialProps`가 있다면 먼저 실행되는 `_app.tsx`의 `getInitialProps`만 실행된다.

  - 만약 페이지 컴포넌트의 `getInitialProps`를 실행하려면 `_app.tsx`를 다음과 같이 변경해야한다.

    ```js
    function MyApp({ Component, pageProps }) {
      return <Component ponent {...pageProps} />;
    }

    MyApp.getInitialProps = async ({ Component, ctx }) => {
      let pageProps = {};
      // 하위 컴포넌트에 getInitialProps가 있다면 추가 (각 개별 컴포넌트에서 사용할 값 추가)
      if (Component.getInitialProps) {
        pageProps = await Component.getInitialProps(ctx);
      }

      // _app에서 props 추가 (모든 컴포넌트에서 공통적으로 사용할 값 추가)
      pageProps = { ...pageProps, posttt: { title: 11111, content: 3333 } };

      return { pageProps };
    };

    export default MyApp;
    ```

<br/>

## shallow route로 화면전환 최적화

같은 페이지 내에서 화면은 같은데 url만 바뀌는 경우, url이 변경될 떄마다 서버에 새로운 페이지를 달라고 요청을 보낸다고 해보자. 그럼 `getInitialProps`나 `getInitialProps`가 매번 실행되어 서버의 연산이 너무 많아지게 된다. 이때 사용할 수 있는 것이 ` shallow route`이다

방법은 아래와 같이 router 객체를 사용하여 쓸 수 있다.

```js
router.push(
  {
    pathname: "/cars?model=bmw",
    query: { ...values, page: 1 }, // 상태값 전달가능
  },
  undefined,
  { shallow: true } // shallow route
);
```

페이지 변경 감지는 `useEffect` 또는 `componentDidUpdate`에서 가능하다

```js
componentDidUpdate(prevProps) {
  const { pathname, query } = this.props.router
  // verify props have changed to avoid an infinite loop
  if (query.counter !== prevProps.router.query.counter) {
    // fetch data based on the new query
  }
}

```

이떄 shallow routing은 동일 페이지의 url 변경에서만 동작하고, 다른 페이지로 이동할 때는 `getInitalProps`가 그대로 실행된다. 다음 예시는 다른 페이지로 이동하는 예시다.

```js
router.push(
  {
    pathname: "/users",
    query: { ...values, page: 1 },
  },
  undefined,
  { shallow: true }
);
```

<br/>

## 출처

https://kyounghwan01.github.io/blog/React/next/getInitialProps/#%E1%84%8C%E1%85%AE%E1%84%8B%E1%85%B4%E1%84%89%E1%85%A1%E1%84%92%E1%85%A1%E1%86%BC

https://nextjs.org/docs/basic-features/data-fetching/get-static-props

https://develogger.kro.kr/blog/LKHcoding/133

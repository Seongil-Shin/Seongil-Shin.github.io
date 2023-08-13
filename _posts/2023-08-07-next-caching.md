---
title: next.js caching
author: 신성일
date: 2023-08-13  22:00:00 +0900
categories: [study, next.js]
tags: [next.js, cache] 
---

next.js app router의 [caching](https://nextjs.org/docs/app/building-your-application/caching)에 관해 공부한 내용입니다.

## Overview

next.js 에서 제공하는 caching mechanism에는 다음과 같은 것들이 있다.

| Mechanism                                                    | What                       | Where  | Purpose                                         | Duration                        |
| ------------------------------------------------------------ | -------------------------- | ------ | ----------------------------------------------- | ------------------------------- |
| [Request Memoization](https://nextjs.org/docs/app/building-your-application/caching#request-memoization) | Return values of functions | Server | Re-use data in a React Component tree           | Per-request lifecycle           |
| [Data Cache](https://nextjs.org/docs/app/building-your-application/caching#data-cache) | Data                       | Server | Store data across user requests and deployments | Persistent (can be revalidated) |
| [Full Route Cache](https://nextjs.org/docs/app/building-your-application/caching#full-route-cache) | HTML and RSC payload       | Server | Reduce rendering cost and improve performance   | Persistent (can be revalidated) |
| [Router Cache](https://nextjs.org/docs/app/building-your-application/caching#router-cache) | RSC Payload                | Client | Reduce server requests on navigation            | User session or time-based      |

next.js는 가능한 많은 캐싱을 적용하려고 하고, 따라서 따로 opt-out하지 않는한 정적으로 렌더링되고, 요청은 캐싱된다. 
각 mechanism은 다음 도식처럼 동작한다.

![Diagram showing the default caching behavior in Next.js for the four mechanisms, with HIT, MISS and SET at build time and when a route is first visited.](/assets/img/2023-08-07-next-caching/image?url=%2Fdocs%2Fdark%2Fcaching-overview.png)

<br/>

## Request Memoization

React에서 확장한 [fetch API](https://nextjs.org/docs/app/building-your-application/caching#fetch)가 자동으로 같은 URL과 옵션으로 보내진 요청의 결과를 기억한다. 따라서 한 route를 렌더링할때 같은 요청이 여러번 발생하면, 딱 한번만 요청을 보내고 그 결과를 여러 곳에서 사용한다.

- Duration : React Component tree의 렌더링이 종료될때까지만 유지. 렌더링 종료시 초기화됨.
- Revalidating : server requests 사이에서 공유되지않으므로 revalidating할 이유가 없음
- Opting-out : `fetch` API에 `AbortController signal`을 줄 수 있음. [예시](https://nextjs.org/docs/app/building-your-application/caching#react-cache-function)
- 기타
  - next.js가 아닌 React 기능임
  - `fetch` 함수의 `GET` method에서만 동작함
  - React Component tree에서만 동작하므로, route handler에서는 동작하지 않음.
  - `fetch`를 쓸 수 없는 환경에서는 [React cache function](https://nextjs.org/docs/app/building-your-application/caching#react-cache-function)을 사용할 수 있음.

<br/>

## Data Cache

여러 server requests와 deployments 사이에서 공유되는 캐시로, next.js에서 확장한 `fetch API`를 사용하면 적용된다. `fetch API`의 `cache`, `next.revalidate` 옵션으로 설정할 수 있으며 자세한 내용은 [fetch API references](https://nextjs.org/docs/app/api-reference/functions/fetch)에서 볼 수 있다.

- Duration : 따로 revlidate하거나 opt-out하지 않는한 계속 유지된다.
- Revalidating
  - Time-based Revalidation : 특정한 시간마다 revalidate 할 수 있음.
    - 동작방식 : `{ next: { revalidate: 60 } }` 일 때,
      - 최초 요청 시 : data source로 API 요청 후, 결과를 캐싱함.
      - 60초 이전 재요청 : 캐시된 데이터 리턴
      - 60초 이후 재요청 : 캐시된 데이터(stale 상태) 리턴, 새로운 데이터 fetch 후 결과 캐싱. **요청이 실패할 경우 이전 데이터(stale)를 계속 유지함.**
  - On-demand Revalidation : 원할때 revalidate 할 수 있음.
    - 동작방식 
      - 최초 요청 시 : data source로 API 요청 후, 결과를 캐싱함.
      - on-demand revalidation 시 : 캐시된 데이터를 `PURGE` 시킴 
      - 재요청 시 : data source로 API 요청 후, 결과를 캐싱함.
- Oping-out
  - `fetch API` 옵션 : `{ cache: 'no-store' }`와 같이 `fetch API` 옵션으로 Data Cache를 사용하지 않을 수 있음.
  - [Route Segment Config options](https://nextjs.org/docs/app/building-your-application/caching#segment-config-options) : 특정 route segment를 기준으로 opt-out 할 수 있음. 해당 route segment에 포함된 모든 data request에 적용된다.
    - `export const dynamic = 'force-dynamic'`
    - `export const revalidate = 0` 
- 기타
  - `{ cache : 'no-store'}`와 같이 캐시를 사용하지 않도록 설정해도 `Request Memoization`은 일어난다. (리액트 동작이기에)

<br/>

## Full Route Cache

next.js는 빌드는 각 route를 렌더링하고 결과를 캐싱한다. 따라서 매 요청마다 렌더링하지 않고 캐시된 route를 응답하여 페이지 로딩을 빠르게 마칠 수 있게한다.

### 동작방식

1. React Rendering on the Server
   - next.js는 React API를 사용하여 렌더링을 오케스트레이션한다. 이때 각 route segments와 Suspense baoundaries를 chunks로 나눈다.
   - 각 chunk는 두 단계로 렌더링된다.
     1. React가 Server Component를 React Server Component Payload라는 스트리밍에 최적화된 특수한 데이터 포맷으로 렌더한다.
     2. next.js는 React Server Component Payload와 Client Component JS instruction을 사용하여 HTML을 서버에서 렌더링한다. 
   - 이로써 작업을 캐싱하거나 응답을 보내기 전에 모든 것이 렌더링될 때까지 기다리지 않고, 작업 완료 시 응답을 스트리밍 할 수 있다.
   - 참고) React Server Component Payload
     - 렌더된 React Server Components tree의 압축 이진 표현. 클라이언트에서 browser's DOM을 업데이트하기 위해 사용된다.
     - 아래 것들을 포함
       - Server Components의 렌더링 결과
       - Client Component가 렌더되고 참조될 위치에 대한 placeholder
       - Server Component에서 Client Component로 내려줄 props
2. Next.js Caching on the Server (Full Route Server)
   - 기본적으로 next.js는 렌더링 결과 (React Server Component Payload, HTML)을 캐싱한다. 빌드타임 또는 revalidation 시 생성되는 결과 캐싱
3. React Hydration and Reconciliation on the Client : 요청 시 클라이언트에서의 동작
   1. HTML 렌더링. 상호작용 불가능
   2. React Server Component Payload를 사용하여 Client와 렌더된 Server Component tree를 조정하고 DOM을 업데이트한다.
   3. JS instruction으로 Client Components를 hydration하고 상호작용을 가능하게 한다.
4. Next.js Caching on the Client (Router Cache)
   - React Server Component Payload는 클라이언트에 Router Cache로서 저장된다. 
   - Router Cache는 navigation 시 이전 방문했던 페이지를 빠르게 렌더링하거나, 다음 route를 prefetching한다.
5. Subsequent Navigations
   - prefetching이나 subsequent navigation 도중 next.js는 React Server Component Payload가 Router Cache에 있는지 검사하고, 있으면 새로운 요청을 server로 보내지 않는다. 

### Static and Dynamic Rendering

route가 빌드타임에 캐시 되는지 안되는지는 그 route가 정적인지 동적으로 렌더링되는지에 달려있다. Static route는 기본적으로 캐시되며, dynamic route는 요청시 렌더링되고 캐시되지 않는다.

### 기타

- Duration : 기본적으로 영구적이며 여러 request에서 재사용된다.
- Invalidations 
  - Revalidating Data : Data Cache를 revalidaing하면 컴포넌트를 서버에서 리렌더링하고, 새로운 렌더링 결과를 캐싱하여 Router Cache도 invalidation 된다.
  - Redeploying : Data cache와는 달리, Full Route Cache는  deployments 사이에서 유지되지 않는다.
- Opting out
  - [Dynamic Function](https://nextjs.org/docs/app/building-your-application/caching#dynamic-functions) 사용 시 Full Route Cache는 opt-out 되고 요청시 렌더링된다. Data Cache는 유지된다.
  - `dynamic = 'force-dynamic'`, `revalidate=0` route segment config 옵션 사용 
    - Full Route Cache, Data Cache 모두 opting-out 한다.
    - Router Cache는 클라이언트 동작이므로 유지된다.
  - Opting out Data Cache : `fetch API`를 캐시하지 않으면 Full Route Cache도 opt-out 된다. 매번 새로운 요청을 보내기 때문이다.

<br/>

## Router Cache

next.js가 클라이언트 사이드에서 가지고 있는 in-memory 캐시이다. React Server Component Payload를 저장하며 각 route segments 별로 분리되어있다. 유저가 routes 사이를 이동할 떄마다 prefetch되거나 방문한 route segment를 저장한다. browser의 bfcache와는 다른 것이지만 비슷한 결과를 낸다. 

- Duration : 브라우저 임시 메모리에 저장되며 다음 두 가지 요인에 의해 duration이 결정된다.
  - Session : Navigation 사이에서 유지되지만, 새로고침 시 사라진다.
  - Automatic Invalidation Period : 개별 segment의 캐시는 특정 시간 이후에 invalidation 된다. 
    - 동적 렌더링 결과물 : 30초
    - 정적 렌더링 결과물 : 5분
    - prefetching 된 동적 렌더링 :  5분까지 저장하도록 설정가능
- Invalidation
  -  [Server Action](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions)에서
    -  On-demand revalidaing 사용 : `revalidatePath`, `revalidateTag`
    - `cookies.set`, `cookies.delete` 사용   (쿠키를 사용하는 route가 stale 되는 것들 방지하기 위해)
  - 클라이언트에서 : `router.refresh`를 사용하면 Router Cache가 전부 clear 됨
- Opting out : Router Cache는 opt-out이 불가능하다. prefetch의 경우는 `<Link>` 컴포넌트에서 props로 opt-out할 수 있지만 방문한 페이지에 대한 캐시는 여전히 남는다.

<br/>

## Cache Interactions

### Data Cache and Full Route Cache

- Data Cache를 opt-out하거나 revalidating 하면 Full Route Cache도 마찬가지로 적용된다. 
- Full Route Cache를 opt-out 한다고 Data Cache는 opt-out 되지 않는다. 한 route 안에 Data Cache가 적용된 요청과 opt-out 된 요청이 공존할 수 있다.

### Data Cache and Client-side Router Cache

- `Route Handler`에서 Data Cache를 revalidating하여도 Router Cache가 바로 invalidate 되지 않는다. Router Handler는 특정 route와 묶여있지 않기 때문이다. Router Cache는 유저가 새로고침하거나 `Automatic Invalidation Period`가 지날때까지 유지된다.
- 즉각적으로 Router Cache를 무효화시키고 싶으면, [Server Action](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions)에서  `revalidatePath`, `revalidateTag`를 사용할 수 있다.

<br/>

## APIs

각 next.js API가 영향을 끼치는 캐시정리

| API                                                          | Router Cache | Full Route Cache      | Data Cache            | React Cache |
| ------------------------------------------------------------ | ------------ | --------------------- | --------------------- | ----------- |
| [`<Link prefetch>`](https://nextjs.org/docs/app/building-your-application/caching#link) | Cache        |                       |                       |             |
| [`router.prefetch`](https://nextjs.org/docs/app/building-your-application/caching#routerprefetch) | Cache        |                       |                       |             |
| [`router.refresh`](https://nextjs.org/docs/app/building-your-application/caching#routerrefresh) | Revalidate   |                       |                       |             |
| [`fetch`](https://nextjs.org/docs/app/building-your-application/caching#fetch) |              |                       | Cache                 | Cache       |
| [`fetch options.cache`](https://nextjs.org/docs/app/building-your-application/caching#fetch-optionscache) |              |                       | Cache or Opt out      |             |
| [`fetch options.next.revalidate`](https://nextjs.org/docs/app/building-your-application/caching#fetch-optionsnextrevalidate) |              | Revalidate            | Revalidate            |             |
| [`fetch options.next.tags`](https://nextjs.org/docs/app/building-your-application/caching#fetch-optionsnexttag-and-revalidatetag) |              | Cache                 | Cache                 |             |
| [`revalidateTag`](https://nextjs.org/docs/app/building-your-application/caching#fetch-optionsnexttag-and-revalidatetag) |              | Revalidate            | Revalidate            |             |
| [`revalidatePath`](https://nextjs.org/docs/app/building-your-application/caching#revalidatepath) |              | Revalidate            | Revalidate            |             |
| [`const revalidate`](https://nextjs.org/docs/app/building-your-application/caching#segment-config-options) |              | Revalidate or Opt out | Revalidate or Opt out |             |
| [`const dynamic`](https://nextjs.org/docs/app/building-your-application/caching#segment-config-options) |              | Cache or Opt out      | Cache or Opt out      |             |
| [`cookies`](https://nextjs.org/docs/app/building-your-application/caching#cookies) | Revalidate   | Opt out               |                       |             |
| [`headers`, `useSearchParams`, `searchParams`](https://nextjs.org/docs/app/building-your-application/caching#dynamic-functions) |              | Opt out               |                       |             |
| [`generateStaticParams`](https://nextjs.org/docs/app/building-your-application/caching#generatestaticparams) |              | Cache                 |                       |             |
| [`React.cache`](https://nextjs.org/docs/app/building-your-application/caching#react-cache-function) |              |                       |                       | Cache       |
| [`unstable_cache`](https://nextjs.org/docs/app/building-your-application/caching#unstable_cache) (Coming Soon) |              |                       |                       |             |

<br/>

## 현재 존재하는 이슈

### route segment config로 opt-out해도 캐시가 적용되는 이슈 

현재 [공식문서](https://nextjs.org/docs/app/building-your-application/caching#segment-config-options)에 따르면 다음 두 설정은 Data Cache과 Full Route Cache를 opt-out 한다고 한다.

- `const dynamic = 'force-dynamic'`
- `const revalidate = 0`

하지만 다음과 같은 이슈를 찾을 수 있었다.

- [Next13 caching fetched data even with the 'revalidate = 0' route segment ](https://github.com/vercel/next.js/issues/51788)

`revalidate = 0`을 주어도 캐시가 적용된다는 이슈이다. 해결방법으로는 다음 두가지가 제시되었지만, 공식적인 답변은 없었다.

- 데이터 변경 후 `router.refresh()`
- `<Link>` 대신 `<a>` 사용

### route segment config와 fetch API의 revalidate 설정 충돌 문제

[next.js 공식문서](https://nextjs.org/docs/app/api-reference/file-conventions/route-segment-config#revalidate)에 따르면, `revalidate` 속성은 한 route에서 가장 낮은 값으로 설정된다고 한다. 그리고 위 설명에서 `revalidate = 0`은 data cache를 opt-out 한다고 하였으니 route segment config로 `revalidate =0`을 설정하면, fetch API에서 `revalidate : 60`을 설정해도 data cache가 무시될 줄 알았다.

하지만 실제로 다음과 같은 코드로 적용해본 결과 data cache가 opt-out 되지 않았다. 캐시가 계속 이루어지고 있었다.

- 코드

```js
// app/layout.js
export const dynamic = "force-dynamic"

export default function Layout({ children }) {
   return (
      <div>
         {children}
      </div>
   )
}
```

```js
// app/post/page.js
export default function Page() {
   const posts = await fetch("http://localhost:8080/api/post", { next : {revalidate:60} });  
   return (
      <div>
         {posts.map(p => <div>{p.content}</div>)}
      </div>
   )
}
```

- next.js 콘솔 결과물

```
 GET /post 200 in 281ms
 │
 ├──── GET http://localhost:8080/api/post 200 in 7ms (cache: HIT)
```

이것이 next.js의 의도된 동작인지, 이슈인지는 좀 더 살펴봐야할 것 같다.

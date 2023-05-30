---
title: next.js - Automatic Static 최적화
author: 신성일
date: 2023-01-29 22:00:00 +0900
categories: [study, next.js]
tags: [next.js]
---





next.js에서는 어떤 페이지가 `getServerSideProps` 또는 `getInitialProps`를 가지고 있지 않다면, static 페이지로 결정한다. 이렇게 static 페이지로 결정되면, 빌드시 그 페이지는 static 페이지로 빌드된다. 

static 페이지는 서버사이드에서 렌더되지 않고 바로 유저에게 전달된다. end-user와 서버 사이에 CDN이 있다면 CDN의 적용 또한 받는다.

### query 객체 처리

`getServerSideProps` 또는 `getInitialProps`가 없는 페이지는 빌드시 prerender 되어 static 페이지가 생성된다. 

prerender 동안은 router의 `query`는 비어있다. 하지만 `hydration`이 완료된 이후 next.js는 페이지를 업데이트하며 `query`객체를 반영한다. 이 과정은 다음과 같은 과정일 때 발생한다.

- 페이지가 [dynamic-route](https://nextjs.org/docs/routing/dynamic-routes)일 경우
- URL에 query 값이 있을 경우
- [Rewrites](https://nextjs.org/docs/api-reference/next.config.js/rewrites)가 `next.config.js`에 설정되어있을 때.

`query`가 완전히 반영되었음을 확인하기 위해선 `next/router`의 `isReady` 필드를 확인하면 된다.

### 빌드 결과물

`next build`로 빌드를 마치면 static 페이지는 `.next/server/pages/about.html`로 결과물이 나온다. 하지만 `getServerSideProps` 을 사용한 페이지는 `.next/server/pages/about.js`로 결과물이 나오게 된다.

### 주의사항

- `getInitialProps`를 사용하는 [Custom App](https://nextjs.org/docs/advanced-features/custom-app)이 있다면, `getStaticProps` 등을 활용한 [Static Generation](https://nextjs.org/docs/basic-features/data-fetching/get-static-props)를 적용하지 않는한 automatic static 최적화가 동작하지 않는다.
- `getInitialProps`를 사용하는 [Custom Document](https://nextjs.org/docs/advanced-features/custom-document)가 있다면, `ctx.req`가 빌드시 정의되어있어야한다. prerender 단계에서 `ctx.req`는 undefined이기 때문이다.
- `next/router`의 `isReady` 필드가 `true`가 되기 전까지 `asPath` 값을 사용하지마라. 





<br/>



## 출처

- [automatic-static-optimization][https://nextjs.org/docs/advanced-features/automatic-static-optimization]

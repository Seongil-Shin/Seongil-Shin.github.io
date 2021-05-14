---
title: [REACT][CORS] 개발 환경에서, 외부 API 와 연결할때 쿠키가 생성되지 않는 문제	
author: 신성일
date: 2021-05-14 16:00:00 +0900
categories: [react]
tags: [react, cors, cookie]
---

# \[REACT]\[CORS] 개발 환경에서, 외부 API 와 연결할때 쿠키가 생성되지 않는 문제

이미 서버에 올라간 백엔드 API 와 로그인 작업을 하던 도중에, 백엔드에서 보내는 쿠키를 브라우저가 저장하지 않는 문제를 발견했다.

찾아보니 CORS 위반으로 생기지 않는 것이라고 하였다.

그럴때 해결방법은 두가지가 있다.

1. package.json에 proxy 설정 추가

```js
//package.json
{
    ...
    "proxy" : "백엔드 주소"
    ...
}
```

위처럼 하면, proxy에서 통신을 하는 것이라고 하여 CORS 를 위반하지 않고 제대로 쿠키를 저장할 수 있다.

2. setuProxy.js

   npm install http-proxy-middleware

   설치후 src 밑에 setupProxy.js 파일을 생성하고 다음과 같이 코드 작성

   ```js
   const { createProxyMiddleware } = require("http-proxy-middleware");

   module.exports = function (app) {
     app.use(
       createProxyMiddleware("/", {
         target: "백엔드 주소",
         changeOrigin: true,
       })
     );
   };
   ```

- 주위점

  위처럼 한다음에 axios 를 호출할땐 다음과 같이 도메인을 제외한 주소를 적어야한다.

  ```js
  await axios.get("/api/product");
  ```

  모든 주소를 적으면, 다른 origin으로 인식하여 다시 쿠키가 생기지 않게 된다

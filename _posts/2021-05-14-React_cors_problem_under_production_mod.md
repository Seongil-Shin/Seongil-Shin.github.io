---
title: REACT/CORS 배포 환경에서, 외부 API 와 연결할때 쿠키가 생성되지 않는 문제
author: 신성일
date: 2021-05-14 16:00:00 +0900
categories: [study, react]
tags: [react, cors, cookie]
---

'배포' 환경에서 CORS 위반으로 쿠키가 생성되지 않을때 다음 2가지 해결방법이 있다.

1. app.js 또는 index.js 에서 axios.defaults.withCredentials = true; 설정 후 axios 호출을 할 때 모든 주소를 적기.

   ```js
   // index.js
   ...
   axios.default.withCredentials = true
   ...

   ----------------
   //productlist.js
   ...
   await axios.get("http://15.123.42.31/api/product")
   ...
   ```

   단, 위처럼 할 경우 개발-배포 환경에서 매번 코드를 변경해줘야하니 불편하다. 따라서 다음처럼 하나의 변수 또는 환경변수를 설정하여 그것을 참조하는 식으로 하면 편하다.

   ```js
   //variable.js
   let base_url = "";
   if(process.env.NODE_ENV === "production") {
       base_url = "http://15.123.42.31"
   }
   export base_url;

   // production.list.js
   await axios.get(`${base_url}/api/product`);
   ```

2. 프록시 서버 사용하기

   사용해본적은 없지만, 프록시 서버를 사용하면 개발-배포환경 상관없이 cors 위반없이 통신이 가능하다고 한다.

   나중에 기회가 생기면 공부하여 포스팅하면 좋을 것 같다.

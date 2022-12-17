---
title: Content-Disposition
author: 신성일
date: 2022-11-11 18:20:26 +0900
categories: [study, web]
tags: [web, header, http]
---


HTTP Response Body에 오는 컨텐츠의 기질/성향을 알려주는 HTTP 헤더 속성이다.

- inline : 디폴트 값. body에 있는 값이 웹 페이지에 표시되어야한다는 뜻.

  `Content-Disposition : inline`

  - 최신 브라우저에서 \<a\> 태그의 `download` 속성은 `Content-Disposition : inline`으로 가져온다.

- attachment : body에 있는 데이터를 다운 받아야한다는 뜻. filename이 명시되어있다면, 해당 filename으로 다운 받는다.

  `Content-Disposition : attachment`

  `Content-Disposition : attachment; filename="filename.jpg"`

- form-data : body가 `multipart/form-datwa`일 경우 적용된다. 데이터가 여러 파트로 나눠져서 보내지도록 해줌. 다음과 같이 name으로 나눠진 필드를 나타낸다.

  ```http
  POST /test.html HTTP/1.1
  Host: example.org
  Content-Type: multipart/form-data;boundary="boundary"
  
  --boundary
  Content-Disposition: form-data; name="field1"
  
  value1
  --boundary
  Content-Disposition: form-data; name="field2"; filename="example.txt"
  
  value2
  --boundary--
  ```

  - name : 가리키는 subpart가 속한 필드의 이름
    - 같은 필드에서 여러 개의 파일을 다룰 때, 여러개의 subpart가 같은 값을 가질 수 있다.
    - 값이 `_charset_` 이면 해당 part가 HTML 필드가 아니라, 디폴트 charset을 사용하겠다는 것을 의미한다.



## **출처**

https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Disposition

https://lannstark.tistory.com/8
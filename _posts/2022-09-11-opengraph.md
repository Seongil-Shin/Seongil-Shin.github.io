---
title: Open Graph(OG) 프로토콜
author: 신성일
date: 2022-09-11 16:00:00 +0900
categories: [study, web]
tags: [web]
---

## Open Graph 프로토콜

HTML문서의 head에는 `<meta>` 태그로 사이트의 정보가 표기되어있는 경우가 많다. 이는 크롤러가 무엇이 제목이고, 무엇이 내용인지 파악하기 쉽도록 하기 위해서 표기한 것이다.

페이스북의 Open Graph 프로토콜은 이러한 메타태그의 표기 방법은 통일한 것이다. 이 Open Graph 프로토콜이 우리가 보는 미리보기 화면의 실체를 구성하는 메타 데이터 표기 방법이다.

<br/>

**작동원리**

1. 사용자가 링크를 입력창에 입력한다.
2. 페이스북, 네이버 블로그, 카카오톡은 입력창의 문자열이 "링크"라는 것을 파악한다.(정규표현식 등으로)
3. 링크라는 것이 파악되면 페이스북, 네이버 블로그, 카카오톡의 크롤러는 미리 그 웹 사이트를 방문해서 HTML head의 오픈그래프 메타데이터를 긁어온다.
4. 이중에서 `og:title`, `og:description`, `og:image`은 각각 제목, 설명, 이미지의 정보를 나타낸다.
5. 그 정보를 바탕으로 미리보기 화면을 생성한다.

<br/>

**기본 메타데이터**

```html
<!-- 제목 -->
<meta property="og:title" content="제목" />
<!-- type -->
<meta property="og:type" content="website" />
<!-- 이미지 -->
<meta property="og:image" content="이미지 경로" />
<!-- canonical link(대표 URL) -->
<meta property="og:url" content="url" />
```

**옵션 메타데이터**

-  og:audio

-  og:description

   -  ```html
      <!-- 설명 -->
      <meta property="og:description" content="내용<br/>줄바꿈" />
      ```

-  og:determiner

-  og:locale

-  og:local:alternate

-  og:site_name

-  og:video

<Br/>

## **트위터의 경우**

트위터는 자사의 메타데이터 표기법을 가지고 있다. 하지만, 웹사이트에 오픈그래프 프로토콜만 있을 경우엔 오픈그래프를 가져온다.

<br/>

## **개발자는 고쳤다는데, 왜 미리보기가 안 바뀔까?**

서버 자원을 아끼기 위해 캐싱된 HTML을 불러오기 때문이다.

페이스북에선 [Sharing Debugger](https://developers.facebook.com/tools/debug/), 카카오스토리에서는 내부 개발자도구를 통해 캐싱 삭제를 할 수 있다.

<br/>

## 클릭 2의 미스터리

내가 바로 쓴 글을 페이스북에 올리면 바로 조회수가 올라간다. 그 이유는 크롤러가 우리 사이트를 한번 미리 방문하기 때문에 조회수 1이 올라가기 때문이다. 구글과 같이 방금 조회한 사람이 크롤러라는 것을 알려주는 경우도 잇지만, 페이스북은 그렇지 않다.

<br/>

## 오픈그래프 적용

1. `og:title`, `og:description`, `og:image` 는 반드시 명시한다.

2. 페이스북의 [Sharing Debugger](https://developers.facebook.com/tools/debug/)를 통해 제대로 적용됐는지 확인한다.
3. 카카오톡, 페이스북, 네이버 블로그 등 사이트마다 이미지가 다르게 보일 수 있다. 따라서 각 사이트에서 제대로 보이는지 확인한다.
4. 가능하다면 A/B 테스트를 통해 어떤 문구가 더 효과적인지 확인한다.

<br/>

## **출처**

https://blog.ab180.co/posts/open-graph-as-a-website-preview

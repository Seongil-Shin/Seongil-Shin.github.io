---
title: Headless CMS
author: 신성일
date: 2023-03-10 23:00:00 +0900
categories: [study, web]
tags: [web, headless, cms]
---



## Headless CMS 란?

웹사이트를 만들 때는 반드시 컨텐츠(데이터)가 필요하다. Headless CMS는 컨텐츠를 보여줄 수단인 Head와 컨텐츠를 분리한 구조로, 컨텐츠가 Head에 독립적으로 동작할 수 있도록 하는 구조이다.

![1*E3qz8MZ8zR7Y3NRghFOvJQ](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*E3qz8MZ8zR7Y3NRghFOvJQ.png)

기존의 CMS는 컨텐츠와 Head가 강하게 묶여있었다. Html에 하드코딩된 문자열들을 생각하면 된다. 

```html
<!-- 기존의 CMS -->
<div>
  content	
</div>
```

이렇게하면 구현은 매우 간단하지만, html과 문자열이 하나의 파일로 강하게 결합되어있다. 

이때 이러한 html이 배포된 상황이라고 해보자. 시간이 흘러 문자열을 수정하고 싶을 때가 온다. 그럼 이 html 파일을 직접 수정하고, 다시 배포하는 과정을 거쳐야한다. 단순히 html 파일 하나만 배포하면 되는 상황이면 큰 문제는 되지않는다. 하지만, 시스템이 복잡해지고 배포과정이 복잡해지면 배포작업은 큰 부담이 된다. 리얼환경 반영까지 오랜 시간이 걸리수도 있다.

하지만 다음과 같이 Headless CMS로 구현하면 상황은 다르다.

```html
<!-- Headless CMS -->
<div id="content">
  <!-- content가 삽입될 곳 -->
</div>
<script>
  const contentDiv = document.getElementById("content")
	fetch("/path-to-api").then((res) => {
    contentDib.innerText = res.content;
  })
</script>
```

데이터베이스, 스토리지 등에 저장된 컨텐츠를 가져와 적절한 위치에 삽입해줌으로써 배포과정없이 변경된 컨텐츠를 반영할 수 있다. 새로운 문자열을 그저 데이터베이스나 스토리지에서 업데이트하면 되기 때문이다.

물론 코드는 복잡해지고, 네트워크 비용이 발생한다는 단점이 있다. 하지만, 자주 변경될 것이라 예상되는 컨텐츠에만 Headless CMS 방식을 적용하면 배포횟수가 줄어들고 결과적으로 개발자는 다른 일에 집중할 수 있게 된다. 또한 수정이 반영되는 속도도 매우 빠르다.

여기서는 배포횟수와 반영속도만을 가지고 장점을 얘기했지만 다음과 같은 장점들도 존재한다.

- Flexibility : 컨텐츠를 하나만 만들면 웹, 앱 등 다양한 채널에 적용이 가능하다. 
- Scalability : Headless CMS는 많은 컨텐츠에 대응이 쉬워서, 성장해가는 서비스에서 적합하다. 
- Security : presentation layer와 결합되어있지 않아 기존의 CMS보다 안전하다. 유저데이터 등 민감한 정보를 코드상에 저장하지 않아도 된다.


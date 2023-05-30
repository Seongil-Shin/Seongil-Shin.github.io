---
title: 프론트엔드 최적화 옵션
author: 신성일
date: 2023-05-30 22:00:00 +0900
categories: [study, web]
tags: [optimization] 
---



프론트엔드에서 최적화를 할 수 있는 선택지를 모아놓은 글이다. 순서는 적용하기 비교적 간편한 순으로 정했다.

# Resource

## 이미지/영상 압축



<br/>

# Javascript

## lodash - debounce, throttle

스크롤, resize 이벤트 콜백함수에서 무거운 작업을 수행하면, 성능에 악영향이 나타날 수 있다. 이때 lodash의 debounce, throttle를 사용하여 이벤트 콜백 호출 횟수를 조절하는 방법이 있다.



<br/>

# CSS

## animation 최적화

1. CSS 애니메이션이 느릴 땐 브라우저 렌더링 과정 중 `Composite` 단계에 속하는 `tranform` 또는 `opacity`로 애니메이션이 동작할 수 있도록 변경한다. 
2. css에 `will-change` 속성을 사용하여 브라우저에게 일어날 변화에 대해 알려줌으로써 최적화할 수 있다.
3. `contain` 속성을 사용하여 해당 엘리먼트의 변화가 다른 dom에 영향을 끼치지 않는다는 걸 명시할 수 있다.

[상세](https://seongil-shin.github.io/posts/css-%EC%B5%9C%EC%A0%81%ED%99%94/#css-%EC%B5%9C%EC%A0%81%ED%99%94)



<br/>

# React

## useMemo, useCallback





<br/>



# Architecture

## SSR 사용

만약 CSR을 사용중이라면 SSR로 전환하여 사용자가 처음 보는 화면을 더 빠르게 띄워줄 수 있다. 

CSR은 리액트가 포함된 자바스크립트를 다운 받고, 리액트를 실행시켜 화면을 렌더링한다. 이때 리액트는 앱 전체를 실행시키기에, 리액트 코드가 방대하다면 그만큼 사용자는 빈 화면에 방치된다.

SSR은 서버에서 미리 첫번째 화면을 생성하고 그 생성된 첫번째 화면의 html, css를 사용자에게 보내는 방법. 사용자 브라우저는 이후 js를 다운 받고 리액트를 실행시켜 인터랙션이 가능하게 한다. 따라서 CSR을 사용하면 사용자가 첫 화면을 보는 시점이 SSR에 비해 늦어지게 된다. 또한 SEO에도 문제가 있다. (검색엔진의 JS 실행에는 한계가 있기에)

SSR에도 단점이 있다. 첫페이지를 구성한 이후 다른 페이지로 넘어갈 때, CSR에서는 그 다른 페이지를 렌더링하기 위한 코드가 이미 있고 필요한 부분만 다시 렌더링하면 되기에 다른 페이지로 넘어갈 때는 CSR이 더 빠르다. 이를 해결하기 위해 CSR와 SSR을 결합하는 방식을 채택한다. (next.js의 [`next/link` 컴포넌트](https://nextjs.org/docs/pages/api-reference/components/script) 참고)

또 기존 리액트에서 SSR에서는 리액트 코드를 입히는 hydration 동작을 앱 전체 단위로 진행시켜서 사용자가 최초로 인터랙션을 하는데까지 시간이 오래 걸렸는데, 이번에 react 18 업데이트에 포함된 Suspense API를 통해 컴포넌트 단위로 hydration을 진행시켜 우선도가 높은 컴포넌트부터 먼저 인터랙션이 가능했다 ([상세](https://seongil-shin.github.io/posts/reactv18/#ssr%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94-%EC%9D%B4%EC%9C%A0))


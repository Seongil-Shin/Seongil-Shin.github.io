---
title: next.js 15 RC
author: 신성일
date: 2024-10-22 19:00:00 +0900
categories: study, article
tags:
  - "#study"
---
https://nextjs.org/blog/next-15

next.js 15의 스테이블버전이 나와 작성하였다

## [Async Request APIs (Breaking Change)](https://nextjs.org/blog/next-15#async-request-apis-breaking-change)

전통적인 SSR 에서는 서버에서는 요청이 전부 완료될때까지 렌더링을 중지한다. 하지만 모든 컴포넌트가 요청이 필요한건 아니다. 따라서 가능한 많은 컴포넌트를 요청이 종료되기전에 준비하기위해, 몇몇 API를 비동기적으로 변경하였다

- cookies, headers, draftMode
- `params` in `layout.js`, `page.js`, `route.js`, `default.js`, `generateMetadata`, and `generateViewport`
- `searchParams` in `page.js`

마이그레이션을 위해 이러한 API를 동기적으로도 접근할 수 있지만, 다음 메이저 버전까지 경고를 발생시킨다. codemod 또는 upgrade guide를 통해 마이그레이션 가능하다
```shell
npx @next/codemod@canary next-async-request-api .
```


## [Caching Semantics](https://nextjs.org/blog/next-15#caching-semantics)

APP Router 에서는 여러 부분에 캐싱이 디폴트로 들어갔다. 하지만 이는 반발도 많아서 `fetch`, `GET` Route Handlers, Client Router Cache를 디폴트로 캐싱되지 않도록 변경하였다


- Client Router Cache no longer caches Page components by default
	- Page 컴포넌트는 `staleTime`을 0으로 설정하여 항상 다시 fetch 하도록 함
	- Shared layout data, back/forward navigation, loading.js 는 캐싱이 적용됨


## [React 19](https://nextjs.org/blog/next-15#react-19)

여러 테스트를 걸친 결과 React 19가 안정적이라고 확신하기에 App Router에서는 React 19 RC를 사용하도록 하였다. 하지만 Pages Route 에서는 React 18 호환성을 제공한다. 추후 React 19가 준비되었을때 업그레이드할 수 있다.

## [Static Route Indicator](https://nextjs.org/blog/next-15#static-route-indicator)

아래와 같은 인디케이터가 개발모드에서 나타나 어떤 라우트가 static 인지 dynamic 인지 확인할 수 있게 해준다. 

![](/assets/images/Pasted%20image%2020241022121849.png)

## [Executing code after a response with `unstable_after` (Experimental)](https://nextjs.org/blog/next-15#executing-code-after-a-response-with-unstable_after-experimental)

로그를 남기거나 분석, 외부 스토리지 싱크 맞추기 등은 유저의 요청과 직접적으로 관련이 없다. 유저의 요청이 이 작업들을 처리하기 위해 기다리지 않도록 `after()`라는 실험적인 API를 추가하였다. 유저에게 응답이 가면 그 후 예약된 작업들을 수행한다

## [`instrumentation.js` (Stable)](https://nextjs.org/blog/next-15#instrumentationjs-stable)

`instrumentation.js` 내 `register()` API 가 이제 stable이다. 
추가적으로 `onRequeestError()` 를 추가하여 에러를 잡을 수 있도록 하였다

## [`<Form>` Component](https://nextjs.org/blog/next-15#form-component)

form 을 다양한 기능과 함께 다룰 수 있는 `<Form>` 컴포넌트를 추가하였다. 

layout, loading UI를 prefetch 하여 제출되었을때 Client-side Navigation을 수행한다. 만약 JS가 로드되지 않았다면 full-page navigation을 수행한다


## [Improvements for self-hosting](https://nextjs.org/blog/next-15#improvements-for-self-hosting)

 `Cache-Control` 을 커스텀하게 설정할 수 있도록 변경되었다

## [Enhanced Security for Server Actions](https://nextjs.org/blog/next-15#enhanced-security-for-server-actions)

기존에는 사용되지 않는 Server Action도 접근이 가능했다. 하지만 이제는 사용되지 않는 Server Action의 ID가 Client-side JS bundle에 포함되지 않도록 변경되었다. 

ID도 빌드할때마다 재계산되어 추측할 수 없도록 변경되었다.


## [Optimizing bundling of external packages (Stable)](https://nextjs.org/blog/next-15#optimizing-bundling-of-external-packages-stable)

외부 패키지를 번들링하는 것은 어플리케이션 cold start 성능을 향상 시킬수 있다

App Router 에서 외부 패키지는 디폴트로 번들되며 `serverExternalPackages` 옵션으로 몇몇 패키지를 뺄수 있다. Pages Router에서는 디폴트로 번들되지 않는데, `transpilePackages` 옵션으로 번들할 패키지를 선택할 수 있다.

이러한 차이를 없애기 위해 `bundlePagesRouterDependencies` 옵션을 추가해서 Page Router에서 디폴트로 번들링하도록 하였다. 이후 `serverExternalPackages`로 제외하고 싶은 패키지를 추가할 수 있다


## [ESLint 9 Support](https://nextjs.org/blog/next-15#eslint-9-support)

ESLint 8의 EOL이 지나 ESlint 9 을 지원한다. 하위 호환성을 맞추었기에 ESLint 8 사용은 계속 가능하다. 


## [Development and Build Improvements](https://nextjs.org/blog/next-15#development-and-build-improvements)

- Server Components HMR
	- save 시 서버컴포넌트를 재요청하여 API 요청이 발생하는 것을 개선하여, HMR 시 `fetch` 응답을 재사용하도록 수정하였다
- Faster Static Generation for the App Router
	- 빌드과정에서 Static Generation 속도를 네트워크 요청과 관련하여 개선하였다 
- Advanced Static Generation Control
	- 더 다양한 요구사항에 대응하기 위해 Static Generation 를 컨트롤할 수 있는 옵션을 추가하였다.
	- 특수한 요구사항이 없다면 그대로 유지하는 것이 좋다. (잠재적으로 OOM, 리소스 과다사용이 발생할 수 있음)


## [Other Changes](https://nextjs.org/blog/next-15#other-changes)

- [Breaking] The minimum required Node.js version has been updated to 18.18.0 
- [Improvement] If a `next/dynamic` component is used during SSR, the chunk will be prefetched 
- [Improvement] The `experimental.allowDevelopmentBuild` option can be used to allow `NODE_ENV=development` with `next build` for debugging purposes
- 
---
title: Vite
author: 신성일
date: 2023-03-21 23:00:00 +0900
categories: [study, react]
tags: [react, vite, webpack, esbuild, rollup]
---

최근 create-react-app 대신에 vite를 공식으로 사용하자는 얘기가 나오고 있다. 

- https://github.com/reactjs/reactjs.org/pull/5487
- [번역본](https://junghan92.medium.com/%EB%B2%88%EC%97%AD-create-react-app-%EA%B6%8C%EC%9E%A5%EC%9D%84-vite%EB%A1%9C-%EB%8C%80%EC%B2%B4-pr-%EB%8C%80%ED%95%9C-dan-abramov%EC%9D%98-%EB%8B%B5%EB%B3%80-3050b5678ac8)

여기서 지적한 문제점들은 다음과 같다

- SSR/SSG 지원의 부족
  - 빈 페이지 로드 -> 리액트 번들 로드 -> 리액트 실행 -> 필요한 데이터 패칭 순으로 이어지는 워터폴 문제 발생
- 모든 앱 코드가 하나의 번들로 묶임
  - 상품 페이지를 로드할때, 장바구니 코드를 로드할 필요는 없는데 CRA는 하나로 묶는다.

리액트 팀에서 답변한 (잠재적인) CRA의 전환 방향은 CRA를 런쳐로 전환하고, 내부적으로는 vite를 사용하는 방법이다. 그러면서 몇가지 리액트 프레임워크를 추천하는 방식이다.

글을 읽으면서 CRA에 대한 이해가 부족하다는 점을 느꼈고, vite가 무엇인지에 대한 지식이 부족하다는 것을 느껴 이 포스트를 작성하게 되었다.

이 포스트에서는 vite에 대해 알아보고, CRA와의 차이점에 대해 작성하였다.

<br/>

## Vite

vite는 빠른 모던 웹 프로젝트 개발 경험을 제공하기 위한 빌드 툴이다. 다음 두가지 메이저 파트로 구성되어있다.

- 개발 서버 : native ES modules를 넘어선 [다양한 기능](https://vitejs.dev/guide/features.html)을 제공한다. (ex : 매우 빠른 [Hot Module Replacement](https://vitejs.dev/guide/features.html))
  - Hot Module Replacement
    - 페이지를 새로고침하거나, 어플리케이션 상태를 날리지 않고 업데이트 된 내용을 즉시 반영한다.
    - `Vue Single File Components`나 `React Fast Refrash` 등 first-party integration을 사용함.
    - `create-vite`를 통해 프로젝트를 생성하면 이미 설정되어있음.
- 번들링 시, [Rollup](https://rollupjs.org/) 기반의 다양한 빌드 커맨드를 사용할 수 있다.

Vite는 다양한 설정이 가장 적절한 값이 디폴트로 설정되어있지만, Plugin API나 Javascript API를 통해 설정이 가능하다. (타입스크립트도 지원)

### Browser Support

디폴트로 설정된 빌드 타켓은 [native ES Modules](https://caniuse.com/es6-module), [native ESM dynamic import](https://caniuse.com/es6-module-dynamic-import), [`import.meta`](https://caniuse.com/mdn-javascript_operators_import_meta)을 지원하는 브라우저다. 하지만, 그 이전 브라우저를 위해 공식으로 [@vitejs/plugin-legacy](https://github.com/vitejs/vite/tree/main/packages/plugin-legacy)을 지원한다.

### vite로  프로젝트 만들기

```sh
npm create vite@latest
# or
yarn create vite
# or
pnpm create vite
```

위 명령어 입력후 프롬프트 지시에 따라 생성할 수 있음. 기본적으로 다양한 템플릿을 제공하고, 필요한 것이 없다면 커뮤니티 템플릿도 사용가능하다.

커뮤니티 템플릿을 사용할 대는 다음과 같이 커맨드창에 입력한다.

```sh
npx degit user/project my-project
cd my-project

npm install
npm run dev
```

<br/>

## create-react-app과의 차이점

### 1. 개발서버의 성능차이

가장 큰 차이는 개발서버에서 vite가 CRA보다 훨씬 빠르다는 것이다.

과거에는 브라우저에서 ESM(ES Module)을 지원하지 않았다. 따라서 javscript 모듈화를 위해서는 번들링이라는 우회방법을 사용해야했다. 이때 사용된 툴이 `Webpack`, `Rollup`, `Parcel`이다. 

하지만 서비스 복잡도에 따라 모듈이 늘어날 수록 개발 서버를 가동하는 것은 매우 오래걸렸다. 따라서 Vite는 브라우저에서 지원하는 ESM 및 네이티브 언어로 작성된 Javascript 도구 등을 이용해 문제를 해결하였다. 

#### 서버구동 최적화

캐시가 없는 상태에서 실행하는 콜드-스타트 방식에서 번들리 기반의 도구는, 애플리케이션 내 모든 소스코드를 확인하고 번들링한다. 하지만, Vite는 다음 두가지로 이 문제를 해결하였다.

- dependencies : 개발 시 내용이 바뀌지 않을 일반적인 javascript 소스코드. vite의 `pre-bundling` 기능은 Esbuild를 사용하여 webpack, parcel 과 같은 기존 번들러 대비 10~100배 빠른 번들링 속도를 보인다.
- source code : 컴파일 과정이 필요하고, 수정이 잦은 코드들. vite는 Native ESM을 이용해 소스코드를 제공한다. 다시 말해 브라우저가 번들러이고, vite는 그저 브라우저의 판단 아래 특정 모듈에 대한 소스코드를 요청하면 이를 전달할 뿐이다. 따라서 조건부 동적 import 이후의 코드는 현재 화면에서 실제로 사용이 되어야만 처리가 된다.

#### 느렸던 소스코드 갱신

기존 번들러 기반에서는 소스코드 업데이트시 번들링 과정을 다시 겪어야했다. 따라서 서비스가 커질수록 소스코드 갱신 시간 또한 증가했다. HMR(Hot Module Replacement)라는 대안이 나왔지만 명확한 해답은 아니었다.

vite는 HMR을 지원하지만, 번들러가 아닌 ESM을 이용해서 제공한다. 어떤 모듈이 수정되는 vite는 그저 수정된 모듈과 관련된 부분만을 교체하고 브라우저에서 해당 모듈을 요청 시 교체된 모듈을 전달할 뿐이다. 따라서 앱 사이즈가 커져도 HMR을 포함한 갱신 시간에는 영향을 끼치지 않는다.

vite는 또한 `HTTP 헤더`를 이용해 퍼포먼스를 높였다. 소스코드는 `304 Not Modified`로, 디펜던시는 `Cache-Control: max-age=31536000,immutable`을 이용해 캐시되도록 함으로써 한번이라도 요청을 덜보내도록 했다.

<br/>

### 2. 번들러의 차이

`create-react-app`으로 만들어진 프로젝트는 `webpack`을 번들러로 사용한다. webpack은 각 모듈을 함수로 감싸 평가한다. 그렇

하지만, vite는 빌드시 `Rollup`이라는 번들러를 사용한다.  Rollup에서는 코드들을 동일한 수준으로 호이스팅하고, 한번에 번들링을 진행하기에 webpack보다 빠르고, 번들링된 결과물도 훨씬 가볍다. 또한 빌드 결과물도 ES6모듈 형태로 출력할 수 있어서 라이브러리나 패키지에 활용할 수 있다.

<br/>

### 3. `index.html`의 위치

vite로 만들어진 리액트 프로젝트는 `index.html`의 위치가 `create-react-app`과 달리 프로젝트 루트에 위치한다. 이는 추가적인 번들링 없이 `index.html` 파일이 앱의 진입점이 되게끔 하기 위함이다.

vite는 `index.html`을 JS 모듈 그래프를 구성하는 하나로 취급한다. 따라서, `<script type="module" src="...">` 태그를 이용해 JavaScript 소스 코드를 가져올 수 있고, 인라인으로 작성된 `<script type="module">`이나 `<link href>`와 같은 CSS 역시 Vite에서 취급 가능하다. 또한 `create-react-app`과는 달리, `index.html` 내에서 url을 표시할 때  `%PUBLIC_URL%`과 같은 placeholder없이 사용할 수 있도록  URL 베이스를 자동으로 맞춰준다.

vite는 또한 정적 HTTP 서버와 비슷하게 루트 디렉터리라는 개념을 가지고 있다. `<root>`라고 언급된 개념이 있는데, 절대경로가 프로젝트 루트를 베이스로 연결된다는 것을 말한다. vite는 또한 프로젝트 루트 밖에 있는 의존성을 가져올 수 있는데, 이로써 모노리포 구성도 가능해진다.

또한 여러 `.html` 파일을 두어  [multi-page apps](https://vitejs.dev/guide/build.html#multi-page-app) 를 구현할 수도 있다.

<br/>

## CRA vs Vite 무엇을 써야할까?

CRA는 vite에 비해 오랜기간동안 사용되어왔고 그만큼 이슈가 많이 픽스되어왔다. 그리고 사용자수또한 CRA의 기반인 webpack이 vite의 기반인 rollup, esbuild보다 훨씬 많다. 그만큼 커뮤니티가 활성화가 되어있고, 개발중 맞닥뜨릴 이슈에 대한 해결책도 많이 제시될 것이다. 

따라서 개인적으로는 기존에 잘 돌아가고 있는 프로젝트라면 현재 개발 서버에 큰 문제가 없을시엔 CRA를 유지하는 것이 좋아보인다. 또한 신규 프로젝트라하더라도 안정성이 중요하다면 CRA를 유지하는 것이 좋을 것같다.

반면 Vite는 기존 프로젝트 중에서 개발 서버의 성능문제가 심각한 경우 또는 신규 프로젝트 중에서 안정성이 상대적으로 덜 중요할 경우 Vite를 사용하는 것이 좋아보인다. (결과적으로 CRA도 vite를 내부적으로 사용하는 것을 생각하고 있다고 하니, vite를 사용하는 것이 트렌드이긴 한가보다.)

<br/>

## 출처

- https://vitejs-kr.github.io/guide/#index-html-and-project-root
- https://junghan92.medium.com/%EB%B2%88%EC%97%AD-create-react-app-%EA%B6%8C%EC%9E%A5%EC%9D%84-vite%EB%A1%9C-%EB%8C%80%EC%B2%B4-pr-%EB%8C%80%ED%95%9C-dan-abramov%EC%9D%98-%EB%8B%B5%EB%B3%80-3050b5678ac8
- https://yoon-dumbo.tistory.com/entry/%EB%A1%A4%EC%97%85%EA%B3%BC-%EC%9B%B9%ED%8C%A9%EC%9D%98-%EC%B0%A8%EC%9D%B4%EC%A0%90-rollup-vs-webpack
---
title: JS bundler, package manager, compiler
author: 신성일
date: 2024-08-31 19:45:24 +0900
categories: study, js
tags:
  - "#study"
---

프론트엔드 관련 아티클을 읽다보면 다양한 bundler, package manger, compiler 도구에 대한 얘기가 나온다. 그때마다 익숙지않은 도구의 이름(pnpm, swc, vite 등)이 나오는데 뭐가 뭔지 잘 이해가 안간다. 따라서 각각의 역할이 무엇인지, 어떤 종류가 있고 어떤 장단이 있는지 간단히 정리해보려고 한다.

## Bundler

여러개의 파일을 하나의 파일로 묶어주는 도구이다. 

초기에 브라우저는 모듈 시스템을 지원하지 않았다. `<script>` 간 모듈을 내보낼 수 없었고, 전역 객체를 통해서만 서로를 참조할 수 있었다. 하지만 웹 생태계가 커지면서 점점 더 다양한 요구조건이 생기기 시작했다. (최적화, 모듈 시스템, 코드 호환성(polyfill), 전처리 등). 번들러는 이러한 요구사항에 대응하기 위해 태어났다.

현재 대표적인 번들러는 `webpack`, `rollup`, `esbuild`, `vite` 이다

webpack
- 초창기에 나타난 번들러로, 지속적인 업데이트를 거쳐가며 많이 사용되어왔다
- "통합"에 집중하여, 번들을 통합하여 관리하도록 하였다
- 장점
	- 풍부한 plugin과 loader
	- 강력한 개발서버 (Hot module replacement 등)
	- Code splitting (파일을 여러 번들로 분리하여 병렬로 스크립트를 로드할 수 있도록 함)

rollup
- module bundler
- "확장"에 집중하여, 작은 코드조각들을 거대하고 복잡한 어플리케이션 혹은 라이브러리로 만들어준다고 소개한다. 
- 같은 소스코드를 다양한 환경에 맞춰 빌드할 수 있기에, 라이브러리를 만들때 큰 도움이 된다
- vs webpack
	- webpack은 내부적으로 commonJS를 사용하고, rollup은 es6(+typescript)를 사용한다
	- 초기에 webpack은 commonJS로만 빌드가 가능했다. rollup은 ES6 모듈형태로 빌드할 수 있기에 이는 라이브러리를 만들때 큰 도움이 되었다.
	- webpack에 비해 좀 더 빠르다. 
		- webpack은 모듈을 함수로 감싸서 평가하지만, rollup은 모듈들을 호이스팅하여 한번에 평가한다
	- 조금 더 가볍다
		- tree shaking은 기본적으로 ES6 코드에서 제대로 동작하기에
	- CommonJS 코드를 ES6 코드에서 사용할 수 있다

esbuild
- `Go`를 기반으로 만들어진 번들러로 성능상 큰 강점이 있다. (JS 기반 번들러보다 10 ~ 100x)
- 아직 메이저 버전이 릴리스 되진 않았지만, 번들러로서 필수적인 기능은 갖췄다
- 하지만 설정이 webpack, rollup 만큼 유연하지 못하고 안정성 이슈가 있다

vite
- esbuild의 단점을 보완한 라이브러리
- `vue.js` 창시자가 만든 `frontend build tool`로, 크게 두가지 기능을 한다
	- `native ES modules` 기반의 강력한 개발서버
	- esbuild로 파일들을 통합하고 rollup을 통한 번들링
- esbuild 로 성능을 높이고, rollup으로 번들링의 유연성을 챙겼다
- vite 프로젝트를 시작하면 설정이 자동으로 된다.
- 자세한 내용은 일전에 [작성한 글](https://seongil-shin.github.io/posts/react-vite/)이 있다
- 키포인트
	- 개발 서버 구동시간이 0에 가까울 정도로 매우 빠름
	- 모든 CommonJS 및 UMD 파일을 ESM으로 불러올 수 있도록 변환함
	- 별도의 설정없이 다양한 리소스 import 가능
	- CSS 빌드 최적화
- 주의할 점
	- 기본적으로 ES6 타겟으로 번들한다. 따라서 ES5 이하를 타겟으로 잡으려면 polyfill 추가가 필요하다. (또는 [@vitejs/plugin-legacy](https://github.com/vitejs/vite/tree/main/packages/plugin-legacy) 플러그인 사용)
	- 기본적으로 `index.html`이 빌드 시작점이고, 순수 JS 번들 생성을 위해서는 라이브러리 모드가 필요하다
- vs cra
	- frontend 개발 도구라는 점에서 CRA와 비교가 많이 된다
	- [자세한 내용](https://junghan92.medium.com/%EB%B2%88%EC%97%AD-create-react-app-%EA%B6%8C%EC%9E%A5%EC%9D%84-vite%EB%A1%9C-%EB%8C%80%EC%B2%B4-pr-%EB%8C%80%ED%95%9C-dan-abramov%EC%9D%98-%EB%8B%B5%EB%B3%80-3050b5678ac8)은 링크된 게시글을 참조하면 좋을거 같고, 리액트 개발 설정을 편하게 해주던 CRA가 여러 단점이 생겼고, vite는 그 단점들을 보완하였다. 성능적으로도 크게 좋아졌다


## Package manager

프로젝트에서 사용하는 패키지를 쉽게 설치, 업데이트해주는 도구이다. 대표적으로는 `npm`, `yarn`, `pnpm`이 있다

npm
- JS 패키지 매니저의 시초이며 node.js에 내장되어 있기 때문에 별도의 설치가 필요없다
- 장점 : 오랜 기간 사용되어왔기에 생태계가 풍부하다
- 단점 : 과거에는 속도, 보안 측면에서 떨어진다는 평가가 있었으나 업데이트를 거치면서 개선되었다

yarn
- npm과 큰 차이가 없고 프로세스도 거의 동일한 패키지매니저
- 장점
	- npm 에 존재하던 보안문제를 해결하여 안전성을 보장하고 있다. 
	- 패키지 설치를 병렬로 진행하여 속도가 빠르다

pnpm
- "performant npm"의 약자로, 효율적인 npm을 표방한다
- 장점
	- npm 에서 node_modules에 매번 패키지를 설치했던 것과 달리 global 저장소에 한번만 저장하여 여러 프로젝트에서 같은 패키지를 참조하는 식으로 저장공간을 절약할 수 있다
	- Ghost Dependency 문제를 해결한다


### npx 는 뭘까?

npm을 좀 더 편리하게 사용할 수 있도록 도와주는 도구로, npm  레지스트리에서 원하는 패키지를 실행할 수 있는 도구이다.

패키지를 설치하지 않더라도 npm 레지스트리에 올라가있는 패키지의 최신버전을 실행 시키고, 자동으로 설치도 된다. 다음과 같은 경우 사용한다.
- npm run-script 를 사용하지 않고 로컬에 설치된 패키지를 사용할 경우
- 일회성 명령으로 실행할 경우
- gist-based scripts를 실행할 경우
- 특정 노드 버전의 스크립트를 실행할 경우


## Compiler

JS는 인터프리터 언어이므로, 일반적으로 쓰이는 컴파일러라는 의미가 아닌 주로 트랜스파일링을 하는 도구를 뜻한다 (Babel, SWC 등)

웹 브라우저 호환을 위해, TS -> JS 변환, 코드 압축 등을 담당한다. 여기서는 간단히 Babel, SWC에 대해서만 조사하였다

Babel
- 주로 최신 자바스크립트 문법을 이전 버전의 브라우저에서도 동작하도록, 이전 문법으로 재작성해주는 역할을 하는 프로그램이다.
- 플러그인이나 프리셋을 통해 기능을 확장할 수 있고, 이를 통해 React나 TS 를 변역할 수도 있다.
- 한계점
	- 배포 뒤 개발자들이 babel이 변경한 코드를 이해하기 어려워할 수 있다
	- 코드의 크기가 늘어난다
	- polyfill을 사용해서 babel이 지원해주지 않는 범위까지 변환해주어야한다
	- 속도가 느리다

SWC
- babel의 단점을 개선하기 위해 next.js 에서 도입한 컴파일러이다
- 장점
	- Rust로 작성하여 속도가 매우 빨라졌다
	- 확장성 : 라이브러리를 설치할 필요없이 next.js에 미리 설치된 SWC를 사용할 수 있다
	- WebAssembly : Rust의 WASM 지원으로 어떤 플랫폼에서도 Next.js 개발을 할 수 있다
- [자세한 내용](https://nextjs.org/docs/architecture/nextjs-compiler#why-swc)은 링크된 게시글 참고


## 참고
- https://bepyan.github.io/blog/2023/bundlers
- https://velog.io/@sebinn/%ED%8C%A8%ED%82%A4%EC%A7%80-%EB%A7%A4%EB%8B%88%EC%A0%80-%EB%B9%84%EA%B5%90-npm-yarn-pnpm
- https://engineering.ab180.co/stories/yarn-to-pnpm
- https://namu.wiki/w/SWC
- https://velog.io/@kwonhygge/Next-JS%EA%B0%80-Babel%EC%9D%84-SWC%EB%A1%9C-%EB%8C%80%EC%B2%B4%ED%95%9C-%EC%9D%B4%EC%9C%A0
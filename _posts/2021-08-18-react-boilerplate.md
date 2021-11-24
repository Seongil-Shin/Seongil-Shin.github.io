---
title: React-boilterplate 설명
author: 신성일
date: 2021-08-18 16:00:00 +0900
categories: [study, react]
tags: [react, boilerplate]
---

## react-boilerplate 설명

- react-boilerplate 라는, 리액트 프로젝트를 처음 시작할때 create-react-app을 대체하여 사용하기에 아주 좋아보이는 프로젝트가 있다.

  https://github.com/react-boilerplate/react-boilerplate

- 상당히 많은 초기 설정을 하는데, 너무 많아서 뭐가 뭔지 알아보기가 어렵다.

- 따라서 이 초기설정들을 설명해놓은 글이 있고, 이 포스트는 이 글을 보고 공부한 내용을 적은 것이다.

  - https://github.com/react-boilerplate/react-boilerplate/blob/master/docs/general/introduction.md

## Tech stack

- Core
  - React
  - React Router
  - Redux
  - Redux Saga
  - Reselect
  - Immer
  - Styled Componentes
- Unit Testing
  - Jest
  - react-testing-library
- Linting
  - ESLint
  - Prettier
  - stylelint

## Project Structure

### app/

- 앱을 개발하는 곳
- container, component 구조를 사용
  - container : redux store랑 연결 되어있는 컴포넌트를 포함
  - components : container에 데이터로 종속된 컴포넌트들을 포함
  - 컨테이너는 어떻게 동작하는지를 정의하고, 컴포넌트는 어떻게 보이는지를 정의함

### internals/

- Configuration, generator, template을 포함
- 앱의 엔진이라고 부를 수도 있음.
- 우리의 소스코드는, 웹팩을 거쳐 브라우저가 이해할 수 있는 형태로 변환된다.
- internals/webpack
  - ECMAScript 6 or ECMAScript 7 를 사용할텐데, 웹백이 주요 브라우저와의 호환성을 맞춰준다.
- internals/generators
  - 새로운 컴포넌트, 컨테이너, 라우트를 스캐폴딩한다
- internals/mocks
  - jest가 테스팅할때 필요한 mocks를 포함함

### server/

- development and production server configuration 파일을 포함

## Basic building Block

### app/app.js 설정

- @babel/polyfill : generator function, Promise 등을 가능하게 함
- history : 브라우저 히스토리를 기억함. ConnectedRouter에서 사용됨
- redux store 가 instantiated 됨
- ReactDOM.render() : \<App /\> 만이 아니라, \<Provider /> \<LanguageProvider /> \<ConenctedRouter /. 또한 렌더함
- Hot module replacement 가 vanilla webpack HMR 를 통해 셋업됨.
  - 모든 reducer, injected sagas, componentes, containers, i18n message를 hot reloadable 가능 하게 함
- i18n internatinalization support setup
- Offline 플러그인이 포함되어 앱을 offine-first하게 함(https://developers.google.com/web/fundamentals/codelabs/offline)
- \<Provider /> 가 redux store와 앱을 연결시킴
- \<LanguageProvider /> 가 language translation을 지원함

### Redux

- configureStore.js에 store 설정이 있음.
- 사용중인 middleware
  - Router middleware : route와 redux store의 동기를 담당
  - Redux saga

### Reselect

- redux state를 슬라이싱하고, 컴포넌트와 관련있는 sub-tree만 제공함.
- Computational power, memoization, composability 라는 세가지 feature를 가지고 있음.
- 자세히는 관련 문서를 봐야할 거 같음.

### Linting

- ESLint, stylelint, Prettier 가 설정되어있어, 세이브할때마다 자동으로 검사해줌
- git hook 또한 설정되어있어서 커밋할때마다 자동으로 코드 상의 에러를 분석/수정 해줌.
- 사용하고 싶지 않으면 package.json의 lint-staged 섹션을 봐라.

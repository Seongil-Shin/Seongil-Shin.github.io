---
title: react 작동원리부터 tailwindcss 사용까지
author: 신성일
date: 2021-12-26 18:19:26 +0900
categories: [study, react]
tags: []
---

## 개요

- yourlist 웹 리뉴얼 프로젝트에서 tailwind css를 사용하기로 결정하였고, create-react-app을 사용하여 리액트앱을 만든 후, tailwind css를 추가하였다.
- 이때 tailwind css는 먼저 css 파일을 컴파일 하고, 컴파일 이후의 css 파일을 import 하는 방식으로 동작한다.
- 따라서 매번 리액트 앱을 실행하기 전에 새로 컴파일을 해줘야하는데, [create-react-app에 tailwindcss 추가하는 가이드](https://tailwindcss.com/docs/guides/create-react-app)를 따라서 하면, 런타임 중에 세이브만으로도 서버를 껐다가 킬 필요없이(hot-reload) tailwind css 가 적용된다.
- 어떤 원리로 위처럼 동작하는지 잘 이해가 되지 않아, react 작동원리부터 차근차근 살펴보기로 하였다.

## Webpack

- 모듈 번들러.
- 개발을 할땐 편의를 위해 하나의 프로그램을 여러 모듈로 나누어 개발한다. 하지만, 브라우저에서 이러한 모듈 시스템은 네트워크의 낭비가 된다. 따라서 여러개의 모듈을 하나의 파일로 묶어서 보낼 모듈 번들러가 필요하고, 그것이 웹팩이다.
- 웹팩에서는 자바스크립트, 스타일시트, 이미지 등 모든 것을 모듈로 봄.
- 웹팩의 중요한 속성
  1. Entry : 웹팩에서 웹 자원을 변환하기 위해 필요한 최초 진입점이자 자바스크립트 파일 경로.
     - 웹팩은 entry를 통해 모듈을 로딩하고, 하나의 파일로 묶는다.
  2. Output : 웹팩에서 entry로 찾은 모듈을 하나로 묶은 결과물을 반환할 위치
  3. loader : 웹팩은 자바스크립트와 JSON만 빌드할 수 있는데, HTML, CSS, IMAGE 를 빌드할 수 있도록 도와주는 속성
  4. Plugin : 웹팩의 기본적 동작 외 추가적인 기능을 제공하는 속성
- 개발 서버를 로컬호스트로 열어주는 역할도 한다.

## react-scripts start

- cra 으로 만들어진 리액트 앱은 웹팩, 바벨 설정을 감추고 react-scripts start 만으로 앱을 브라우저에 띄워주며, hot reloading 기능을 제공해준다.
- 과연 react-script start를 실행하면 어떤일이 벌어지는 걸까?

### 과정

1. 먼저 Webpack 이 실행되어 src/index.js 부터 살펴본다. 그리고 import 된 모듈을 따라가며 dependency graph를 전부다 살펴볼때까지 진행한다.

2. 이후 모든 자바스크립트 파일들을 Babel로 넘겨주어 ES5+ 로 문법을 번역한다.

3. 웹팩은 모든 자바스크립트 파일을 하나의 파일로 만들고, 서버를 실행한다.

4. 이후 코드가 변경되고 저장이 되면, webpack에 연결된 hot-reload 모듈을 통해 다시 1번부터 과정을 반복한다.

## postCSS

- 위 과정까지 보았다면, 리액트 앱 또한 매번 컴파일 과정을 다시 반복하여 다시 서버를 여는 것을 알 수있다.
- 또 개요에서, tailwindCSS를 사용하기 위해선 앱이 실행되기 전에 먼저 CSS 파일을 컴파일 해야한다는 것을 말했다.
- 그럼 tailwindCSS를 사용하면서 기존에 리액트를 개발하던 것과 같이 hot raload를 사용하려면, react-scripts start 과정의 1번을 시작하기 전에 tailwindCSS를 자동으로 컴파일 하면 된다는 것을 알 수 있다.
- 과연 어떻게하면 될까? 여기서 postCSS가 등장한다.

### postCSS

- JS 플러그인을 사용하여 CSS를 변환시키는 CSS postprocessor이다. CSS의 babel과 같은 것이다.

- postprocessor이므로, preprocessor인 SASS와 다르게, CSS를 processing하여 플러그인을 거쳐 새로운 css로 변환해준다.

  ```
  CSS -> Parser -> Plugin -> Stringfier -> New CSS
  ```

  - 단, postCSS 자체는 아무런 일을 하지 않고, 플러그인이 모든 변환작업을 한다.

### postcss.config.js

- 개요에 소개한 가이드를 따라가다보면, 프로젝트에 postcss.config.js 파일이 추가됨을 확인할 수 있다. 파일 내용은 다음과 같다.

```js
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};
```

- 위 코드를 보면 tailwindcss 플러그인과, autoprefixer 플러그인이 추가됐음을 알 수 있다.
  - tailwindcss : tailwind css를 컴파일 해주는 플러그인
  - autoprefixer : 브라우저별 호환성을 맞춰주도록 css를 변환시켜주는 플러그인.

## Creat-react-app v5

- 자, 그럼 가이드를 따라가면 어떻게 hot reload가 되는걸까?

### 기존의 방법

- tailwindcss를 CRA에서 사용하는 방법을 검색해보면 수많은 포스트에서 CRACO를 사용해야한다고 나와있다.

  - CRACO : CRA의 설정(웹팩 등)을 npm eject를 하지 않더라도 변경해줄수 있도록 하는 라이브러리

- 이 CRACO을 통해 postcss를 webpack의 hot-reload 과정에 등록하여 hot-reload가 되는 방식으로 하였다.

### Creact-react-app v5

- 개요의 가이드를 보면, CRA v5 로 만들어진 프로젝트를 기반으로 한다. 그리고 CRA v5 의 업데이트 목록에는 다음과 같은 tailwind support가 추가되었다고 한다.
  - https://github.com/facebook/create-react-app/pull/11717
- 위 PR은, tailwind.config.js 파일이 있으면 이를 불러들여 자동으로 CRA의 webpack config를 변경하여 tailwind를 지원하도록 하는 내용이다.
- 그리고 개요에 나와있는 'npx tailwindcss init -p' 를 실행하면, tailwind.config.js 이 자동으로 생성된다.
- 따라서 이러한 방식을 통해 개요의 가이드만을 따라가도, 다른 여러 복잡한 설정을 거치지 않아도 hot-reload가 가능한 것이다.

## 의문

- 그럼 왜 postcss.config.js 파일을 생성하는 걸까?
- CRA v5에서도 tailwind.config.js 가 필요하다. 이를 위해선 npx tailwindcss init -p 가 필요하고, 이 작업을 통해 postcss.config.js도 추가 된다.

- 이때 "npx tailwindcss init -p" 를 사용하는 프로젝트에는 CRA v5로 만들어지지 않은 프로젝트도 존재할테니, postcss.config.js도 추가되도록 놔둔 것으로 생각된다.

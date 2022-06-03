---
title: React 18로 업데이트 방법, 문제점 정리
author: 신성일
date: 2022-06-03 14:00:00 +0900
categories: [projects, forkie]
tags: []
---

기존에 React 17을 사용하던 Forkie Player 프로젝트를 React 18로 업데이트 하는 기록

## 업데이트 하는 이유

1. `automatic batching`
   - 동영상이 끝났을때 다음 영상으로 넘어가거나, 현재 영상의 시작부분으로 돌아가기 위해선 동영상의 timeupdate, ready, pause, ended 등등의 상태에 대응할 필요가 있다.
   - 이때 이벤트 핸들러 안에서 여러 상태업데이트를 할때, automatic batching이 없을때는 서로 다른 타이밍에 발생하는 업데이트를 대응하기가 까다로웠다.
   - react 18로 업데이트하면서 automatic batching이 적용되고 이러한 문제점이 사라지길 기대하며 업데이트를 하였다.
2. `Suspense`

   - 현재 forkie에서는 SSR를 사용하지 않기 때문에, SSR을 위해 도입한 것은 아니다.
   - 하지만 `검색`, `플레이리스트 수정` 등을 할때 Loading일 경우 fallback을 하도록 하는 좀 더 깔끔한 코드를 원했기 때문에 react 18을 도입하였다

3. 항상 신기술을 원하는 개발자의 마음...
   - 새로운 것이 나오면 **어쨌든 더 좋지 않겠어?** 라는 마음으로 설치하게 되어버린다.
   - 만약 새로운 툴에 문제가 있다면 설치하지 않는 것이 좋으나, 페이스북에서 공들이고있는 React이고, 상당히 긴 알파/베타 기간을 거친 걸로 알고 있는데, 충분히 stable하다고 생각되어서 업데이트를 결정하였다.

<br/>

## 업데이트 방법

공식문서에 나온대로 업데이트 하면 된다.

https://reactjs.org/blog/2022/03/08/react-18-upgrade-guide.html

<br/>

<br/>

## 업데이트 과정 중 문제점

### typescript 에러

다른 분이 작성한 [문서](https://velog.io/@seungmini/TypeScript%EC%97%90-React18-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0)를 보며 해결하였다.

<br/>

### 패키지 호환성문제

타입스트립트를 사용하지 않을 경우 큰 문제는 없다. 하지만, 타입스크립트를 사용할 경우 패키지와 사용하는 리액트 버전이 차이가 나기에 타입 에러가 발생할 수도 있다.

자주 업데이트가 되는 패키지의 경우 이미 react 18을 대응하여 업데이트를 해놨을테니 해당 버전으로 업데이트를 하면된다.

하지만 문제가 되는 경우가 두가지 있다.

1. 새로운 버전에 큰 변화가 있어 앱에서 해당 패키지를 사용하는 부분을 함께 수정해야하는 경우
2. 자주 업데이트되는 패키지가 아니어서, react 18을 대응한 버전이 없을 경우

위 두가지 문제를 해결하기 위해선 `force-resolution`을 하여야한다. 패키지 매니저로 npm을 사용하는 경우와 yarn을 사용하는 경우 다르게 대응하여야한다. 이 글은 yarn을 기준으로 적었다.

yarn 에서는 `package.json`에 `resolution` 을 추가함으로써 해결할 수 있다고 [공식문서](https://classic.yarnpkg.com/lang/en/docs/selective-version-resolutions/)에서 말하고 있다.

forkie에서 적용한 예시는 다음과 같다.

```json
{
  "name": "forkie-player_web_renewal",
  "dependencies": {
    "@types/react": "^18.0.10",
    "@types/react-dom": "^18.0.5",
    "react": "^18.1.0",
    "react-dom": "^18.1.0"
  },
  "resolutions": {
    "react": "^18.1.0",
    "react-dom": "^18.1.0",
    "@types/react": "^18.0.10",
    "@types/react-dom": "^18.0.5"
  }
}
```

이후 `yarn install`을 하면 된다.

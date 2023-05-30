---
title: 성능 최적화를 위한 next.js case study
author: 신성일
date: 2023-03-27 23:00:00 +0900
categories: [study, next.js]
tags: []
---



[원글](https://www.patterns.dev/posts/nextjs-casestudy)에서는 [TMDB](https://www.themoviedb.org/)의 클라이언트 웹 어플리케이션을 만들고, 여러번의 실험을 거쳐 성능 최적화에 달성하는 과정을 서술하였습니다.

이 글은 기본적으로 원글을 학습하면서 제가 읽기 편한 방식으로 번역/정리한 글입니다. 정확한 내용을 알고 싶으시면 원글을 보시는 걸 추천드립니다.

<br/>

## 용어 정의

- FCP : First Contentful Paint. 사용자가 화면에서 콘텐츠를 볼 수 있는 페이지 로드 타임라인의 첫 번째 지점. 
- LCP : Largest Contentful Paint. 뷰포트에서 가장 큰 콘텐츠 엘리먼트가 나타날때 까지 걸린 시간
- TTI : Time To Interactive. 상호작용까지의 시간
- TBT : Total Blocking Time. 총 차단 시간. 메인 스레드가 입력 응답을 막을 만큼 오래 차단되었을 때, FCP와 TTI 사이 총 시간을 측정
- CLS : Cumulative Layout Shift

## 사용 툴

- Lighthouse
- WPT : https://www.webpagetest.org/webvitals

<br/>

## Packages Switched

앱을 만들 때 사용한 서드파티 리액트 컴포넌트를 분석하여, 무겁거나 메인 스레드를 blocking 하는 컴포넌트들은 다른 컴포넌트로 대체하였다. 그 결과로 다양한 메트릭에서 좋은 결과를 냈다.

- `@svgr/webpack`을 `Font-awsome` 대신에 사용하니, Speed Index는 34%, LCP는 23%, TBT는 51% 개선되었다.
- `react-burger-menu`을 대체하고 `resize-sicky-box`로부터 resize-observer-polyfill을 제거하고 custom-built component를 사용하였다. 이는 번들 사이즈가 34.28KB 감소하는 결과로 이어졌다.
- `React Select` 대신 `React Select Search`를 사용하니 LCP 35%, CLS 100%, TBT 18% 개선되었다.
- `React Slick` 대신 `React Glider`를 사용하여 TBT 77% 개선되었다.
- 브라우저 지원을 위해 `native smooth scrolling` 대신에 `React Scrolling`을 사용하였다.
- `React Rating` 대신에 `React Stars`를 사용하여 TBT가 33% 개선되었다.

그럼 상세한 과정을 알아보자

### SVG icon Library

처음에 편리함과 유명세 때문에 Font-Awesome 라이브러리를 사용하였다. 하지만, 라이브러리 로딩시 큰 transfer size 때문에 웹 페이지 로딩 속도에 악영향을 주어 Lighthouse 점수가 안좋게 나왔다.

따라서 `@svgr/webpack`으로 대체하고, 라이브러리 전체가 아닌 개별 아이콘을 import 하는 방식을 통해 성능을 개선하였다.

```js
import { FontAwesomeIcon } from "@fortawesome/react-fontawesome";
 
// replaced by
 
import HeartIcon from "public/assets/svgs/icons/heart.svg";
import PollIcon from "public/assets/svgs/icons/poll.svg";
import CalendarIcon from "public/assets/svgs/icons/calendar.svg";
import DotCircleIcon from "public/assets/svgs/icons/dot-circle.svg";
```

### Application menu

처음에 react-burger-menu를 사용하였으나, 분석해보니 번들 사이즈가 너무 컸다. 다양한 커스터마이징에 대응하기 위해 CSS 스타일이나 애니메이션이 많았기 때문이다. 하지만 그정도의 기능이 필요없었기에 커스텀하여 메뉴를 구현하였다. 이는 번들사이즈를 매우 크게 줄여주었다.

### Dropdown for Sort feature

유저가 정렬 옵션을 선택하여 정렬하기 위해 `react-select`를 처음에 사용하였다. 하지만 `react-select`에서 제공하는 것만큼 다양한 기능은 필요없었고, 단순한 정렬 컴포넌트만 원했다. 따라서 `react-select-search` 컴포넌트를 사용하였다. 이로써 번들 사이즈카 크게 줄었고, lighthout 점수도 올랐다.

### Cast Carousel

출연배우를 보여주기 위한 수평 glide를 위해 `React-Slick`을 사용하였으나, `react-glider`로 대체하여 번들 사이즈를 크게 줄였다.

### The Scrollling Component

앱은 페이지네이션이 구현되어있다. 그리고 이전 페이지, 이후 페이지로 이동할 때마다 페이지의 상단으로 스크롤된다. 이 스크롤을 부드럽게 하기 위해 native smooth scroll 기능을 사용하였다.

```js
window.scroll({
  top: 0,
  left: 0,
  behavior: "smooth",
});
 
// and
 
document.querySelector(`.${SCROLL_TO_ELEMENT}`)?.scrollIntoView({
  behavior: "smooth",
});
```

하지만 이 기능은 모든 브라우저에서 지원하진 않았다. 따라서 `react-scroll` 패키지를 사용하여 대응하였다. 이 패키지는 성능에 조그마한 악영향을 주었지만 같은 스크롤 기능을 구현하였다.

### The Rating Component

`React-rating`은 동그라미, 별, 따봉 등 다양한 심볼로 커스터마이징하여 레이팅을 매길 수 있게 해준다. 하지만, 테스트 앱에서는 별만을 사용하기로 하였기에  `react-stars`로 대체하였다. 이로써 번들사이즈는 줄었으나 크게 줄진 않았지만, `React-rating`이 SVG 아이콘을 사용한 것에 비해, `react-stars`는 ★ 심볼을 사용하여 성능에 크게 영향을 주었다. Lighthouse 검사 결과 TBT가 33% 줄었다.

<br/>

## 다른 기술들

### Code-Splitting

유저가 정말 필요할때만 Menu가 보이도록 Menu 컴포넌트를 lazy-loading 하기 위해 코드를 분리하였다. 따라서 페이지 로딩 후 메뉴 컴포넌트 로딩을 보장하는 [LazyLoadingErrorBoundary](https://github.com/googlechromelabs/adaptive-loading/commit/1309d73ed5e773aa84574bf3959ebbe227594d85)를 사용하였다. 
이로써 FCP, LCP는 그대로지만, TBT는 71% 개선되었다.

### Inline the Critical, Defer the Non-Critical

CSS는 render-blocking 리소스이다. 페이지가 렌더되기 전에 반드시 로드되고 프로세스되어야한다. 일부 CSS는 최초 페이지에 보여야한다. 이것들은 Critical CSS이다. 하지만 천천히 로드해도 되는 CSS들도 있다. 

next.js 문서에 따라 node module CSS 파일을 `/pages/_app.js`에 import 했다. `react-glider`와 `react-modal-video`가 CSS import를 요구했기 떄문이다. 하지만 _app.js에 CSS import 하는 것은 render-blocking을 일으킨다. 하지만, 해당 CSS들은 모든 페이지에서 필요한 것이 아니다.

따라서 이 컴포넌들이 사용하는 CSS는 컴포넌트가 사용되는 곳에 inline으로 처리하였다. 

```jsx
<div ref="{ref}" className="cast">
  <Glider
    hasArrows
    slidesToShow="{slidesToShow}"
    slidesToScroll="{1}"
    itemWidth="{GLIDER_ITEM_WIDTH}"
  >
    {cast.map(person => (
    <PersonLink key="{person.id}" person="{person}" baseUrl="{baseUrl}" />
    ))}
  </Glider>
</div>
<style jsx>
  {`
  /*CSS Classes required for Glider*/
  `}
</style>
```

이 변화로, FCP, LCP, TTI에 2~5%의 성능향상이 있었다. 전체 성능도 79%에서 81%로 증가하였다.

### Aspect Ratio for Images

Lighthouse에서 CLS에 대한 문제점을 짚어주었다. 어떤 이미지가 CLS을 일으키는지도 알려주었다. 3G와 같은 환경에서는 특히 문제가 될 수 있기에 해결하기로 했다.

[aspect-ratio-boxes 기법](https://css-tricks.com/aspect-ratio-boxes/)를 사용하여 이미지의 aspect ratio를 지정하였다. 이로써 페이지가 로딩중이라도 이미지가 요구하는 공간을 충분히 확보하여, 이미지가 로드되었을 때 CLS가 일어나지 않도록 하였다. 이로써 CLS가 0.016에서 0으로 줄어들었다.

테스트 앱을 만든 후 [CSS aspect ratio](https://developer.mozilla.org/en-US/docs/Web/CSS/aspect-ratio)의 브라우저 호환성이 향상되어 잘 작동한다. 따라서 이 기능을 사용하는 것도 좋다.

### Preconnects

 [preconnects](https://web.dev/preconnect-and-dns-prefetch/#:~:text=A%20benefit%20of%20specifying%20a,that%20hosts%20the%20font%20files)를 사용하여 브라우저에게 어떤 리소스를 사용할 것인지 힌트를 줄 수 있다. `rel=preconnect`를 설정하면 브라우저에게 해당 도메인과 연결을 맺으라고 알려주어 이후 과정을 빠르게 할 수 있다. 이로써 리소스를 빠르게 가져올 수 있다.

```html
<link rel="preconnect" href="https://image.tmdb.org" />
<link rel="preconnect" href="https://api.themoviedb.org" />
```

차이는 작지만 다음과 같이 조금 더 빨라진 모습을 볼 수 있다. 

| Performance Metric | FCP  | Speed Index | LCP   | TTI   | TBT   |
| ------------------ | ---- | ----------- | ----- | ----- | ----- |
| Before             | 0.9  | 3.9         | 3.43  | 2.93  | 60    |
| After              | 0.83 | 3.5         | 2.86  | 2.63  | 53.33 |
| % Change           | 7.77 | 10.25       | 16.61 | 10.23 | 6.67  |

### Optimize the API call sequence

여러 API를 호출할 때, 화면을 그리는데 필요한 API는 다른 API에 의해 늦춰지면 안된다.  따라서 다음과 같은 개선를 줄 수 있다.

| **BEFORE**                                                   | **AFTER**                                       |
| :----------------------------------------------------------- | ----------------------------------------------- |
| 장르나 설정같은 메타데이터를 불러오기 위한 API 호출 때문에 영화 포스터를 가져오는 API 요청이 put off 됨 | 메타데이터와 영화 포스터를 동시에 가져오도록 함 |
| 영화 포스터 데이터를 fetch함                                 | 홈화면을 영화 포스터 데이터와 함께 렌더함       |

### Preloading API response

유저가 처음 접속했을 때, `Popular` 장르와 첫번째 페이지를 보여준다는 것이 정해져있음. 따라서 처음 접속한 유저의 API 요청 옵션은 다음과 같음
`Genre="popular" page=1`

이에 기반하여 홈화면에 사용할 데이터를 preload할 수 있다.

```html
<link
  rel="preload"
  as="fetch"
  href="https://api.themoviedb.org/3/movie/popular?api_key=844dba0bfd8f3a4f3799f6130ef9e335&page=1"
  crossorigin="true"
/>
```

하지만 이 데이터가 사용되지 않으면 불필요한 네트워크 비용이 발생하니 잘 사용하자. 

### Preloading the logo and the TMDB trademark

모든 페이지에서 보이는 logo와 TMDB tademark를 preloading하여 FCP와 Speed Index를 5~6% 향상시켰다.

```html
<link
  rel="preload"
  href="{LOGO_IMAGE_PATH}"
  as="image"
  media="(min-width: 80em)"
/>
<link
  rel="preload"
  href="{DARK_TMDB_IMAGE_PATH}"
  as="image"
  media="(prefers-color-scheme: dark) and (min-width: 80em)"
/>
<link
  rel="preload"
  href="{LIGHT_TMDB_IMAGE_PATH}"
  as="image"
  media="(prefers-color-scheme: light) and (min-width: 80em)"
/>
```

<br/>

## Making the site Responsive

[SSR과 반응형 디자인을 섞는건 어려움](https://nitayneeman.com/posts/combining-server-side-rendering-and-responsive-design-using-react/)이 있다. 

- 서버는 클라이언트의 `window` 객체를 모른다. 따라서 `window.matchmedia()`와 같은 메서드는 사용할 수 없다. 거기다 [client hints](https://caniuse.com/?search=client%20hint) 또한 모든 브라우저에서 지원되는 것은 아니다.
- CSS media query는 클라이언트가 데스크톱인지 모바일인지 상관하지 않고 렌더링된다.

이러한 어려움을 극복하기 위해 `@artsy/fresnel` 라이브러리를 사용하였다. 이로써 CSS breakpoint로 DOM의 모든 엘리먼트를 서버에서 렌더할 수 있다. 그리고 그 breakpoint에 일치하는 컴포넌트만 마운트된다. 이 방식으로, 중복된 마크업과 불필요한 렌더링을 방지했다. 

| **PERFORMANCE PARAMETER** | **FCP (S)** | **SPEED INDEX (S)** | **LCP (S)** | **TTI (S)** | **TBT (MS)** |
| ------------------------- | ----------- | ------------------- | ----------- | ----------- | ------------ |
| Before                    | 0.93        | 3.73                | 2.6         | 2.63        | 60           |
| After                     | 1.06        | 3.23                | 2.66        | 2.66        | 63.33        |
| % Change                  | 13.97       | 13.4                | 2.3         | 1.14        | 5.55         |

`@artsy/fresnel` 번들이 포함되어 몇몇 지수에서는 성능이 악화되었으나, Speed Index는 빨라졌다. 마크업 코드를 줄일 수 있는 것도 좋은 trade-off 이다.

<br/>

## Ideas that did not help

Lighthouse의 피드백에 기반하여 몇가지 대책을 세웠으나 성능상 이점이 없었던 아이디어들 모음

1. `react-lazyload` 패키지 대체
   - `react-lazyload` 패키지를 lazy loading image를 위해 사용하고 있었다. 이 패키지는 메인 스레드를 오래 사용하였다. 이것을 [native iamge lazy-loading](https://addyosmani.com/blog/lazy-loading/)으로 대체하려 했다.
   - 그러나 TBT는 11배 증가한반면 LCP는 근소하게 감소했다. 이는 native image lazy loading은 뷰포트에 가까운 몇개의 이미지를 로드하고, `react-lazyload`는 뷰포트 안에 있는 이미지만을 로드하기 때문일 수 있다.  
   - 최근에는 Next.js Image Component를 사용할 수도 있다.
2. image dimension을 설정
   - `Aspect Ratio for Images`를 세팅하기 전에 CLS를 향상시키기 위해 image dimension을 설정하였다. 
   - 하지만 aspect ratio 만큼 잘 동작하지 않았다.
3. SSR
   - LCP를 감소시키기위해 SSR을 하였으나, 향상보단 퇴화가 있었다. 
   - 이유는 페이지를 렌더링할때 필요한 영화 데이터와 이미지가 TMDB API를 통해 가져와지기 때문이다. 이는 서버 응답을 느리고 만들었다. 왜냐만 모든 API 요청/응답이 서버에서 이루어졌기 때문이다.

<br/>

## Ideas that might help

1. responsive image를 preloading과 함께 구현 [링크](https://web.dev/preload-responsive-images/)
2. [service worker를 이용한 캐싱](https://web.dev/learn/pwa/caching/)
3. 리덕스를 포함하여 부풀어진 _app.js 개선. landing일때 리덕스가 필요없는 페이지들도 있기 때문. 코드 분리를 통해 개선
4. 리덕스 없이 SSR 구현, [SSR Caching](https://github.com/vercel/next.js/tree/canary/examples/ssr-caching) 사용
5. 더 가벼운 패키지로 교체
6. 서드파티 라이브러리를 로드하기 위해 [React loading pattern](https://www.patterns.dev/posts/nextjs-casestudy)을 사용하여 lazy-loading/code-splitting 기법 적용
7. 히어로 사진과 같이 최초 몇개 이미지를 위해 [Image post-processing](https://github.com/vercel/next.js/pull/15875) 적용
8. SVG loading spinner를 CSS animation으로 대체
9. 내부적으로 Javscript를 사용하는 `<Image />` 와 달리, HTML/CSS로 이루어진 가벼운 컴포넌트 사용

<br/>

## 정리

성능 개선을 위해서 다음과 같은 사항을 고려해볼 수 있다.

### 패키지 변경

현재 사용하고 있는 패키지에 불필요한 기능까지 포함하고 있는지, 번들 사이즈는 어느정도인지, 상세 구현이 어떻게 되어있는지(SVG? symbol?) 등을 따져봐서 패키지를 좀 더 가볍고 빠른 것으로 대체할 수 있다.

### 코드 분리

유저가 최초로 랜딩할 페이지에서 필요없는 자원들을 가져오지 않도록 적절히 코드/리소스를 분리한다.

### 네트워크 요청 우선순위

API, 이미지 등을 불러올 때, 랜딩 페이지에서 우선적으로 보여야하는 것들을 다른 것들보다 느리게 불러오지 않도록 한다.ㄴ

### CLS

CLS를 줄이기 위해 aspect ratio를 지정하거나, Next.js의 Image 컴포넌트를 사용할 수 있다.

<br/>

## 출처

https://www.patterns.dev/posts/nextjs-casestudy

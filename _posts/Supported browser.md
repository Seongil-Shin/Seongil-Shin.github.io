next.js 공식문서 항목 중 [Supported Browser](https://nextjs.org/docs/architecture/supported-browsers) 항목이 있다. 지원하는 브라우저에 관한 항목인데, 직접 경험한 바로는 ES5가 지원되는 환경(IE11 포함)에서는 대체로 돌아가는 편이었다. 하지만 공식적으로 언급한 부분이 있으니 잘 숙지하여 문제를 겪지 않도록 하는 것이 중요하다.
만약, 호환성에 맞지않는 코드를 작성하면 다음과 같은 에러가 발생하며 앱이 동작하지 않게 된다.

![Pasted image 20230819181043.png](/assets/image/Pasted image 20230819181043.png)

## Browserlist

next.js에서 공식적으로 명시한 지원되는 브라우저는 다음과 같다. 

```json
  
{ 
	"browserslist": [ 
		"chrome 64", "edge 79", "firefox 67", "opera 51", "safari 12" 
	]
}

```

## Polyfills

next.js에서 디폴트로 지원하는 polyfill은 [링크](https://github.com/vercel/next.js/blob/canary/packages/next-polyfill-nomodule/src/index.js)에 나와있다. 웬만하면 링크에 나와있는 polyfill이 지원되는 API만 사용하되, 꼭 사용해야하는 API가 있다면 custom polyfill을 사용할 수 있다.

Custom Polyfill을 추가하기 위해서는 사용하려는 컴포넌트에서 직접 import 할 수도 있고, top-level 컴포넌트에서 import 해줄 수도 있다.

## JavaScript Language Features

ES6 기능에 더해 다음과 같은 기능을 지원한다.
- [Async/await](https://github.com/tc39/ecmascript-asyncawait) (ES2017)
- [Object Rest/Spread Properties](https://github.com/tc39/proposal-object-rest-spread) (ES2018)
- [Dynamic `import()`](https://github.com/tc39/proposal-dynamic-import) (ES2020)
- [Optional Chaining](https://github.com/tc39/proposal-optional-chaining) (ES2020)
- [Nullish Coalescing](https://github.com/tc39/proposal-nullish-coalescing) (ES2020)
- [Class Fields](https://github.com/tc39/proposal-class-fields) and [Static Properties](https://github.com/tc39/proposal-static-class-features) (part of stage 3 proposal)

## Server-side Polyfills

`fetch()` 의 경우 polyfill을 제공한다. 하지만 구형 서버에서 node.js 버전을 올릴 수 없는 상황에서 next.js를 사용해야하는 경우를 제외하곤 server-side 는 보통은 걱정하지 않아도 된다. 

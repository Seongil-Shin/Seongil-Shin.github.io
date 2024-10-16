---
title: the state of es5 on the web
author: 신성일
date: 2024-10-15 19:00:00 +0900
categories: study, js
tags:
  - "#study"
---
https://philipwalton.com/articles/the-state-of-es5-on-the-web/

ES5의 현재 상태와 웹 성능 및 코드 호환성에 미치는 영향을 다루는 글.

ES5 코드를 작성하는 건 문제가 아님. 하지만 ES6+ 코드를 작성했을때 이를 ES5로 트랜스파일하는 것은 많은 Polyfill을 포함하게 함. 대부분의 사이트에서 ES5로 트랜스파일을 시키고 있으므로, 이는 상당한 낭비를 발생시키고 있음.


## The Data

- Bundler and build tool's default cofiguratios
	- 대부분의 번들러에서는 ES5로의 트랜스파일을 디폴트로 설정하고 있지 않고 모던 브라우저를 지원하는 추세로 가고 있음.
	- 하지만 가장 많이 쓰이는 Babel은 ES5로의 트랜스파일이 디폴트임.
- Popular JavaScript libraries
	- `date-fns`, `three.js` 등은 ES6+ 문법을 포함하고 있음. 이는 Babel 등의 트랜스파일러에서는 라이브러리까지는 트랜스파일 하지않으니 최종 결과물에도 ES6+ 문법이 포함되게 됨
- ES5 usage in the wild
	- 상위 웹사이트 중 89%가 ES6+ 문법을 보유중
	- 79%는 ES5 헬퍼 코드 포함, 68%는 ES5 헬퍼코드와 ES6+ 코드를 모두 포함함
	- ES5 헬퍼코드와 ES6+ 코드를 공존시킬 의미가 없는데도 68% 그러고 있다. 그 이유는 아래 두가지 중 하나일 것이다
		- ES5 호환성이 필요없는데, 어떤 라이브러리에서 포함시키고 있는 걸 수 있음
		- ES5 호환을 맞추려고 했는데, 생각지도 못하게 몇몇 라이브러리에서 ES6+ 코드를 포함시킴

데이터들을 보면 대부분의 웹사이트에서 불필요한 ES5 헬퍼코드를 서빙하고 있고, 이는 ES5 호환이 대부분의 비지니스에서는 불필요하다는 것을 시사한다.



## Recommendations

- 라이브러리 개발자
	- 89% 의 웹사이트가 ES6+ 문법을 그대로 사용중이므로 굳이 ES5로 변환하는 과정을 거치지 않아도 됨. 라이브러리 사용자가 더 넓은 커버리지를 원할수도 있지만, 이는 사용자가 알아서 선택해서 트랜스파일해야하는 것이 좋음
	- [baseline](https://web.dev/baseline?hl=ko), [webdx](https://www.w3.org/community/webdx/) 에서 참고하여 적절한 지원범위를 선택하는 것이 좋다.
	- `Baseline Widely Available` 을 타겟팅하는 것은 과거에 머무르지 않게 해준다. 아래처럼 browserlist 쿼리를 작성할 수도 있다
```js
targets: [ 
	'chrome >0 and last 2.5 years', 
	'edge >0 and last 2.5 years', 
	'safari >0 and last 2.5 years', 
	'firefox >0 and last 2.5 years', 
	'and_chr >0 and last 2.5 years', 
	'and_ff >0 and last 2.5 years', 
	'ios >0 and last 2.5 years', 
]
```

- 웹사이트 개발자
	- 많은 웹사이트에서 ES6+랑 ES5 헬퍼가 혼재한다는 것은 트랜스파일 과정에서 `node_modules`를 제외하는건 그닥 좋은 방법이 아니라는 것을 시사함 (개발자가 타게팅한 버전에 맞지 않는 코드가 모듈에 있을 수 있기에)
	- 따라서 개발과정에서는 `node_modules`를 제외하되 배포과정에서는 포함하는 방법을 생각해볼수 있다


## 최종 결론

- 현재 대부분의 사이트에서는 IE11 (ES5) 호환성을 맞출 필요성이 없다
- Build tool은 고정된 버전의 브라우저를 지원하지 않아도 된다.  `Baseline Widely Available`을 타겟팅하여 오래된 기능을 자연스럽게 미지원하는 것이 바람직하다
- 웹사이트 개발자들은 빌드 과정에 라이브러리를 트랜스파일링하는 것을 포함시켜야한다. 라이브러리가 해당 웹사이트가 목표로하는 버전을 못맞추고 있을 수 있기 때문이다
- cross-browser 지원은 빌드툴에만 의존하지 않는 것이 좋다. 직접 테스트를 해보자


ts는 target: es5로 되어있는데?
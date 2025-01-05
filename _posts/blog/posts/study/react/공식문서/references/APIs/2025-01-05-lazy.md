https://ko.react.dev/reference/react/lazy

`lazy`를 통해 로딩 중인 컴포넌트가 처음으로 렌더링 될때까지 연기할 수 있다


## Reference

**lazy(load)**

```js
import { lazy } from "react"

const MarkdownPreview = lazy(() => import("./MarkdownPreview.js"));

function Markdown() {
	return (
		<Suspense fallback={<Loading />}>
			<MarkdownPreview />
		</Suspense>
	)
}
```
- load : Promise나 thenable(`then` 메서드가 있는 Promise 유사 객체) 을 반환하는 함수. React는 반환된 컴포넌트를 처음 렌더링하려고 하기 전까지는 `load`를 호출하지 않는다. 렌더링 하려고 할때 React는 먼저 `load`를 실행한 후 `load`가 이행될 때까지 기다렸다가 이행된 값의 `.default`를 React 컴포넌트로 렌더링한다. 반환된 Promise와 Promise의 이행된 값이 모두 캐시되므로 `load`를 두번 호출하지는 않는다.

- 반환값 : 트리에 렌더링할 수 있는 React 컴포넌트를 반환한다. 컴포넌트가 로드되는 동안 렌더링을 시도하면 일시 중지된다. 이때 로딩 인디케이터를 보여주려면 `<Suspense>`를 사용할 수 있다.

## 주의

아래와 같이 컴포넌트 내부에서 사용할 경우 리렌더링 시 항상 다시 불러오게 된다. 따라서 항상 모듈의 최상위 수준에서 선언하여야한다.

```js
function Markdown() {
	// 주의
	const MarkdownPreview = lazy(() => import("./MarkdownPreview.js"));
	
	return (
		<Suspense fallback={<Loading />}>
			<MarkdownPreview />
		</Suspense>
	)
}
```
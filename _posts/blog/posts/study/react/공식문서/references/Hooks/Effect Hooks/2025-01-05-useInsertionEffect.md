`useEffect`의 버전 중 하나로, DOM 변이 전에 실행된다
> `useInsertionEffect`는 CSS-in-JS 라이브러리를 위한 훅이다. CSS-in-JS 라이브러리 작업 중에 스타일을 주입하고자 하는 경우가 아니라면, `useEffect`나 `useLayoutEffect`가 더 나을 수 있다.


```js
import { useInsertionEffect } from 'react';  

// Inside your CSS-in-JS library  
function useCSS(rule) {  
	useInsertionEffect(() => {  
		// ... inject <style> tags here ...  
		// ... <style> 태그를 여기에 주입합니다.  
	});  
	return rule;  
}
```

### parameter
- setup
- dependencies (optional)

### Caveats
- Effect는 클라이언트에서만 실행되며 서버렌더링 중에는 실행되지 않는다.
- `useInsertionEffect` 내부에서는 state를 업데이트할 수 없다.
- `useInsertionEffect`가 실행될때까지는 ref가 첨부되지않았고, DOM이 업데이트 되지 않았다.
- 스타일 주입은 `useInsertionEffect`에서 사용하는 것이 좋은데, `useInsertionEffect`에서 스타일을 주입하고 다른 Effect를 실행할 때는 업데이트 된 스타일을 보지만, 다른 Effect에서 스타일을 주입하면 주입 전 스타일을 보기에 레이아웃 계산이 잘못될 수 있다.


## Usage

### Injecting dynamic styles from CSS-in-JS libraries
CSS-in-JS에는 다음 세가지 방법이 있다.
1. 컴파일러를 사용하여 CSS 파일로 정적 추출
2. 인라인 스타일 (`<div style={{ opacity: 1 }} >`)
3. 런타임에 `<style>` 태그 주입

일반적으로 1, 2번 방법이 추천된다. 3번은 다음 이유로 권장되지 않는다.
1. 런타임 주입은 브라우저에서 스타일을 훨씬 더 자주 다시 계산하도록 한다.
2. 런타임 주입이 React 라이프사이클에서 잘못된 시점에 발생하면 속도가 매우 느려질 수 있다.

첫번째 문제는 해결할 수 없지만, 두번째는 `useInsertionEffect`로 해결 가능하다
```js
// Inside your CSS-in-JS library  
let isInserted = new Set();  

function useCSS(rule) {  
	useInsertionEffect(() => {  
		// As explained earlier, we don't recommend runtime injection of <style> tags.  
		// But if you have to do it, then it's important to do in useInsertionEffect.  
		if (!isInserted.has(rule)) {  
			isInserted.add(rule);  
			document.head.appendChild(getStyleForRule(rule));  
		}  
	});  
	return rule;  
}  

function Button() {  
	const className = useCSS('...');  
	return <div className={className} />;  
}
```


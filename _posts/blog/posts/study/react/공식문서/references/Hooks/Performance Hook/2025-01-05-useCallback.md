https://react-ko.dev/reference/react/useCallback
리렌더링 사이에 함수 정의를 캐시할 수 있게 해주는 React Hook

```js
const handleSubmit = useCallback((orderDetails) => {  
	post('/product/' + productId + '/buy', {  
		referrer,  
		orderDetails,  
	});  
}, [productId, referrer]);
```

### Caveats
- `useCallback`은 성능 최적화를 위해 사용해야한다. `useCallback` 없이 코드가 동작하지 않는다면 근본적인 문제를 찾아 수정해야한다.
- `useCallback`으로 함수를 캐싱하는 건 몇가지 경우에만 유용하다
	- `memo`로 감싼 컴포넌트에 prop으로 전달하는 경우.
	- 함수가 다른 훅의 dependencies로 사용되는 경우
- 몇 가지 원칙으로 `memoization`이 불필요해질 수 있다
	- 컴포넌트가 다른 컴포넌트를 시각적으로 감쌀 때, JSX를 children으로 받아들여라. 
		- wrapper 컴포넌트가 변경되더라도 자식이 리렌더링 되지 않음
	- local state를 선호하고, 필요이상으로 state를 끌어올리지마라.
		- ex) 최상위 트리나 전역 state에 아이템이 호버되었는지와 같은 일시적 state를 두지마라
	- 렌더링 로직을 순수하게 유지하라.
		- 컴포넌트를 리렌더링했을 때 문제가 발생하면 버그가 있는 것이므로, memoization 대신 버그를 수정하라
	- [state를 업데이트하는 불필요한 Effect](obsidian://open?vault=Obsidian%20Vault&file=react%2F%EC%8B%A0%20%EA%B3%B5%EC%8B%9D%EB%AC%B8%EC%84%9C%2FEscape%20Hatches%2FYou%20might%20not%20need%20an%20Effect)를 피하라
		- React 앱의 대부분의 성능 문제는 컴포넌트를 반복해서 렌더링하게 만드는 Effect에서 발생하는 업데이트 체인으로 인해 발생
	- [Effect에서 불필요한 의존성을 제거](https://react-ko.dev/learn/removing-effect-dependencies)하라
		- memoization 대신 object나 함수를 Effect 내부나 컴포넌트 외부로 이동시키는게 더 간단할 수도 있음.

## Usage

### Optimizing a custom Hook
커스텀 훅을 사용하는 경우에 반환하는 모든 함수를 `useCallback`으로 감싸는 것이 좋다.
```js
function useRouter() {  
	const { dispatch } = useContext(RouterStateContext);  
	
	const navigate = useCallback((url) => {  
		dispatch({ type: 'navigate', url });  
	}, [dispatch]);  
	
	const goBack = useCallback(() => {  
		dispatch({ type: 'back' });  
	}, [dispatch]);  
	
	return {  
		navigate,  
		goBack,  
	};  
}
```
이렇게하면 훅의 소비자가 필요할 때 자신의 코드를 최적화할 수 있다.




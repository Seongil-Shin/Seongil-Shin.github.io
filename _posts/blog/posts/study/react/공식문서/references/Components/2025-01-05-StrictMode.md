https://react-ko.dev/reference/react/StrictMode
개발 중인 컴포넌트에서 흔히 발생하는 버그를 조기에 발견할 수 있도록 해준다.
```jsx
<StrictMode>  
	<App />  
</StrictMode>
```

- 불완전한 렌더링으로 인한 버그를 찾기위해 한번 더 렌더링한다
	- 리액트는 모든 컴포넌트가 순수하다고 가정하기에, 두번 렌더링함으로써 개발자로 하여금 이상을 발견할 수 있도록 함
	- 다음을 두번 호출함
		- 컴포넌트 함수 본문
		- `useState`, `set 함수`, `useMemo`, `useReducer
- Effect 클린업이 누락되어 발생한 버그를 찾기위해 Effect를 한번 더 실행한다.
- 지원 중단된 API의 사용여부를 확인한다.
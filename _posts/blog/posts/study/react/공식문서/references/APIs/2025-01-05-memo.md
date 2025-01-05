https://react-ko.dev/reference/react/memo

컴포넌트의 props가 변경되지 않은 경우 리렌더링을 건너 뛸 수 있다.

```jsx
const MemoizedComponent = memo(SomeComponent, arePropsEqual?)
```


### Props
- `Component` : 메모화 하려는 컴포넌트
- `arePropsEqual`(optional) : 컴포넌트의 이전 prop 및 새로운 prop의 두 인자를 받는 함수. 
	- 새로운 prop과 이전 prop이 같으면 true를 반환해야한다. 
	- 일반적으로는 `Object.is`로 적용되고, 따로 지정하진 않는다.
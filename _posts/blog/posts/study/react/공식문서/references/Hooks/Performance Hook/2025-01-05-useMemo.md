https://react-ko.dev/reference/react/useMemo

리렌더링 사이에 계산 결과를 캐시할 수 있는 React Hook

```js
const cachedValue = useMemo(calculateValue, dependencies)
```


### Parameter
- `calculateValue`
	- 캐시하려는 값을 계산하는 함수. 
		- 순수함수 / 인자를 받지 않음 / 어떤 타입이든 반환해야함
	- React는 초기 렌더링 중에 호출하고, 이후 `dependencies`가 변화되지 않으면 동일한 값을 반환한다.
- `dependencies`
	- `calculateValue` 에서 참조하는 모든 반응형 값들의 목록. 
		- props, state, 컴포넌트 본문 내에서 선언된 모든 변수와 함수

### Returns
- 초기 렌더링에서 인자없이 `calculateValue`를 호출한 결과를 반환함.
- 이후 `dependencies`가 변화하지 않으면 같은 값 반환


### Caveats
- Strict Mode 에서는 `calculateValue`를 두 번 호출한다
- React는 특별한 이유가 있지 않는 한 캐시된 값을 유지하려고 한다
	- 유지 안하는 예시
		- 개발 모드에서 컴포넌트 파일을 수정했을 시
		- 초기 마운트 중에 컴포넌트가 일시 중단 되었을때
	- 향후 React에서는 캐시 폐기를 좀 더 활용할 수도 있다
		- ex) virtual list를 기본지원한다면, view port에서 벗어난 요소를 캐시에서 폐기하는 등

### tips

- useMemo를 통한 성능 최적화는 몇 가지 경우에만 유용하다
	- `useMemo`를 넣는 계산이 눈에 띄게 느리고 의존성이 거의 변하지 않는 경우
	- `memo`로 감싼 컴포넌트에 props으로 전달되는 경우
		- memo를 사용해도, object나 함수를 전달하면 매번 참조값이 달라지기에 매번 리렌더링된다.
		- 하지만, useMemo를 사용하면, `dependencies`가 변경되지 않는한 리렌더링을 건너띌 수 있다.
	- 다른 hook(useEffect, useMemo 등)의 dependencies에 포함되어있을 경우
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


https://react-ko.dev/learn/lifecycle-of-reactive-effects


- effect는 컴포넌트가 마운트 된 상황에서 여러번 실행하고, clean up을 수행할 수 있다.
	- 의존하는 props, state가 변경될때 clean up 수행 후 새로운 props, state로 effect 수행
- effect는 다음과 같은 경우 수행한다.
	- 마운트 되었을 때 -> effect 수행
	- dependencies에 있는 값이 변경된 경우 -> 이전 값으로 clean up  수행, 새로운 값으로 effect 수행
	- 언마운트 되었을때 -> clearn up 함수 수행
- effect는 reactive value에만 반응한다
	- 렌더링 시 변경되는 값 (state, props)에만 반응한다.
	- `serverUrl`과 같은 상수 값은 dependencies에 넣어도 의미없다.
- 컴포넌트 본문에서 선언된 모든 변수는 reactive하다
	- props와 state만 reactive value가 아니라 이들로부터 계산되는 값 역시 반응형이다. 
- `useRef`에서 반환된 ref 객체는 리렌더링 시 변경되지 않도록 보장되는 안정적인 값이다. 따라서 dependencies에서 제외해도 된다.
- dependency를 선택할 수는 없고, effect에서 읽은 모든 reactive value를 포함하여야한다.
	- linter에서 넣으라는 값을 넣지 않아야하는 경우는 코드를 변경해야한다는 뜻
- 객체나 함수를 의존성으로 사용하면 안된다. 매 렌더링마다 새로 만들어져 effect가 매 렌더링 이후에 실행된다.
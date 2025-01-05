- 구 가이드 : https://ko.legacy.reactjs.org/docs/strict-mode.html
- 신 가이드 : https://react.dev/reference/react/StrictMode

## 요약
`StrictMode`는 애플리케이션 내의 잠재적인 문제를 알아내기 위한 도구입니다.

- StrictMode는 개발 모드에서만 활성되며 프로덕션 빌드에는 영향을 끼치지 않는다.
- 앱의 어디서나 활성화가 가능하며, `StrictMode` 컴포넌트 아래 컴포넌트에만 영향을 준다.
- `Fragment`와 같이 UI 렌더링에 영향을 주지 않는다.

## 검사하는 사항

1. 안전하지 않은 생명주기 메서드를 사용하는 컴포넌트 발견
2. 레거시 문자열 ref 사용에 대한 경고
3. 권장되지 않는 findDOMNode 사용에 대한 경고
	- 주어진 클래스 인스터스를 바탕으로 트리를 탐색해 DOM 노드를 찾을 수 있는 메서드로, 부모가 특정 자식이 렌더링되는 것을 요구하는 상황이 허용되어 추상화 레벨이 무너지게 되었다.
	- 이로인해 부모가 자식의 DOM 노드에까지 닿을 가능성이 있어 컴포넌트의 세세한 구현을 변경할 수 없게 되어 리팩토링이 어려워지는 상황을 만들었다.
4. 예상치못한 부작용 검사
	- React는 렌더링-커밋 두 단계로 동작하는데, 리액트는 커밋하기 전에 렌더링 단계의 생명주기 메서드를 여러번 호출하거나 아예 커밋을 하지 않을 수도 있다.
		- 렌더링 단계 생명주기 메서드
			- constructor
			- componentWillMount
			- componentWillReceiveProps
			- componentWillUpdate
			- getDerivedStateFromProps
			- shouldComponentUpdate
			- render
			- setState
	- 이 메서드들은 여러번 호출 될 수 있기때문에 부작용이 발생하면 메모리 누수 등 문제를 일으킬 수 있다. 
	- StrictMode를 통해서 아래 메서드들을 의도적으로 이중으로 호출하여 문제를 예측가능하게 도와준다.
		- 클래스 컴포넌트의 `constructor`, `render`, `shouldComponentUpdate`
		- 클래스 컴포넌트의 `getDerivedStateFromProps`
		- 함수 컴포넌트 바디
		- State updater 함수 (`setState`의 첫번째 인자)
		- `useState`, `useMemo`, `useReducer`에 전달되는 함수
5. 레거시 context API 검사
6. Ensuring reusable state
	- 미래 기능으로 이전의 상태를 유지하면서 컴포넌트를 추가하거나 제거하는 기능이 추가된다. 
	- React 18에서는 여기에 맞추어, StrictMode에 컴포넌트가 상태를 유지하며 처음으로 마운트 되었을때 자동으로 언마운트 후 다시 마운트하도록 하여 이상 동작이 없는지 체크한다.
- 
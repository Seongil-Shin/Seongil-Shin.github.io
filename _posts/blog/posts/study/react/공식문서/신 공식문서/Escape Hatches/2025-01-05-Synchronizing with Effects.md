https://react-ko.dev/learn/synchronizing-with-effectss

Effect로 특정 이벤트가 아닌 렌더링 자체로 발생하는 사이드 이펙트를 명시할 수 있다.
Effect는 화면 업데이트 후 [커밋](obsidian://open?vault=Obsidian%20Vault&file=react%2F%EC%8B%A0%20%EA%B3%B5%EC%8B%9D%EB%AC%B8%EC%84%9C%2FAdding%20interactivity%2FRender%20and%20Commit)이 끝날 때 실행되며 이때가 React 컴포넌트를 일부 외부 시스템과 동기화하기에 좋은 시기이다.

## You might not need an Effect

[Effect가 필요없을 수도 있다.](obsidian://open?vault=Obsidian%20Vault&file=react%2F%EC%8B%A0%20%EA%B3%B5%EC%8B%9D%EB%AC%B8%EC%84%9C%2FEscape%20Hatches%2FYou%20might%20not%20need%20an%20Effect) Effect는 일반적으로 React 외부 시스템과 동기화하는데 사용한다는 점을 명심하자. (ex : 브라우저 API, 서드파티 위젯, 네트워크 등)
외부시스템이 없고 다른 state를 기반으로 일부 state만 조정하려는 경우 Effect가 필요하지 않을 수 있다.

## 의존성 지정

- 의존성 배열을 넣지 않았을 때 : 매 렌더링 이후 실행됨
- 빈 배열을 넣었을 때 : 최초 렌더링 이후 실행됨
- 의존성을 넣었을 때 : 렌더링 이후 Object.is()로 검사하여 달라졌을 경우 실행

`ref` 객체는 넣어야할까?
- React는 렌더링할 때마다 동일한 useRef 호출에서 항상 동일한 객체를 얻을 수 있도록 보장한다.
- 절대 변하지 않으므로 그 자체로 Effect가 다시 실행되지 않는다. 따라서 포함여부는 중요하지 않다.
- 다만 ref가 변화할 수 있는 경우에는 포함해야한다. (props로 건내받는 경우 등)


## 개발 환경에서는 Effect가 두번 씩 실행된다.

React는 개발환경에서 버그를 찾기 위해 컴포넌트를 의도적으로 다시 마운트한다.
	- 클린업 함수가 없는 경우
	- 무한 루프가 발생하는 경우
production 모드에서는 한번만 실행된다.


## Effect가 아닌 애플리케이션 초기화 함수

다음과 같이 컴포넌트 외부에 넣으면 브라우저가 페이지를 로드한 후 한번만 실행된다.
```js
if (typeof window !== 'undefined') { // 실행환경이 브라우저인지 여부 확인  
	checkAuthToken();  
	loadDataFromLocalStorage();  
}  

function App() {  
	// ...  
}
```

## 클린업 함수가 실행되는 시점

React는 항상 다음 렌더링의 Effect 전에 이전 렌더링의 클린업 함수를 실행한다.


## 각 Effect는 해당 렌더링의 값을 캡처한다.

Effect 안에서 `text`라는 상태를 사용하면, 각 렌더링마다 Effect는 분리되어있고 각 Effect는 해당 렌더링 시점의 `text` 값을 가진다. 

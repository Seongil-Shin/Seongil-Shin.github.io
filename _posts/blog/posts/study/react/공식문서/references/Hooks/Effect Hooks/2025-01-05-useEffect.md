https://react-ko.dev/reference/react/useEffect
컴포넌트를 외부 시스템과 동기화할 수 있는 React Hook

> 외부시스템 : 브라우저 API, 네트워크 등 리액트로 제어되지 않는 코드 조각

### parameters

- `setup`
	- Effect의 로직이 포함된 함수. 클린업 함수를 반환할 수 있다.
	- 컴포넌트가 DOM에 추가되면 셋업 함수를 실행한다.
	- 의존성이 변경되어 리렌더링 될 때마다 React는 먼저 이전 값으로 클린업 함수를 실행한다음, 새 값으로 셋업 함수를 실행한다.
	- 컴포넌트가 DOM에서 제거되면, React는 마지막으로 클린업 함수를 실행한다.
- `dependencies` (optional)
	- setup 함수 내에서 참조된 모든 [반응형 값](obsidian://open?vault=Obsidian%20Vault&file=react%2F%EC%8B%A0%20%EA%B3%B5%EC%8B%9D%EB%AC%B8%EC%84%9C%2FEscape%20Hatches%2FSeparating%20Events%20from%20Effects)(props, state, 컴포넌트 본문 내 선언된 함수/변수)의 목록
	- React는 각 의존성에 대해 Object.is로 이전 값과 비교한다. 
	- 빈배열 vs 아예 넣지 않았을때
		- 빈배열 : 컴포넌트가 처음 DOM에 추가되었을때만 실행함
		- 아예 넣지 않았을때 : 매 리렌더링마다 실행함

### Caveats
- `useEffect`는 Hook이므로 컴포넌트 최상위 레벨 또는 커스텀 훅에서만 호출 가능하다. 반복문/조건문 안에서는 호출할 수 없고, 따라서 필요시 새 컴포넌트로 추출하라
- 외부 시스템과의 동기화 목적이 아니면 [Effect가 필요없을 수 있다.](obsidian://open?vault=Obsidian%20Vault&file=react%2F%EC%8B%A0%20%EA%B3%B5%EC%8B%9D%EB%AC%B8%EC%84%9C%2FEscape%20Hatches%2FYou%20might%20not%20need%20an%20Effect)
- Strict Mode가 켜져있으면 첫 번째 셋업 시 셋업 + 클린업을 한번 더 실행한다.
- 의존성 배열에 컴포넌트 내부에 정의된 객체/함수가 포함되면 Effect가 필요이상으로 자주 실행될 수 있다. 
- Effect가 상호작용(클릭 등)으로 인한 것이 아니라면, React는 브라우저가 Effect를 실행하기 전에 화면을 먼저 업데이트한다. 따라서 Effect가 UI 작업을 하고 있거나 지연이 눈에 띄면 [useLayoutEffect](obsidian://open?vault=Obsidian%20Vault&file=react%2Freferences%2FHooks%2FEffect%20Hooks%2FuseLayoutEffect)를 사용할 수 있다.
- Effect는 클라이언트에서만 실행되고 서버렌더링 중에는 실행되지 않는다.


## Usage
### Controlling a non-React widget
외부 시스템을 컴포넌트의 특정 prop이나 state와 동기화하고 싶을 때가 있다. 
아래 예제는 컴포넌트 내 `zoomLevel` prop과 외부 Map 컴포넌트와 동기화 시키는 예제이다.
```js
import { useRef, useEffect } from 'react';
import { MapWidget } from './map-widget.js';

export default function Map({ zoomLevel }) {
  const containerRef = useRef(null);
  const mapRef = useRef(null);

  useEffect(() => {
    if (mapRef.current === null) {
      mapRef.current = new MapWidget(containerRef.current);
    }

    const map = mapRef.current;
    map.setZoom(zoomLevel);
  }, [zoomLevel]);
  ...
}
```
`MapWidget` 클래스는 자신에게 전달된 DOM 노드만 관리하기 때문에 클린업 함수가 필요하지 않는다. `Map` 리액트 컴포넌트가 트리에서 제거되면 DOM 노드와 `MapWidget` 클래스 인스턴스는 브라우저 JS 엔진에 의해 GC된다.

### Fetching data with Effects

Effect 내부에서 데이터 패칭할 경우, 다음과 같이 클린업 함수에서 플래그를 지정하면 네트워크 응답이 보낸 순서와 다르게 도착해도 race condition이 발생하지 않는다.

```js
import { useState, useEffect } from 'react';
import { fetchBio } from './api.js';

export default function Page() {
  const [person, setPerson] = useState('Alice');
  const [bio, setBio] = useState(null);
  
  useEffect(() => {
    async function startFetching() {
      setBio(null);
      const result = await fetchBio(person);
      if (!ignore) {
        setBio(result);
      }
    }

    let ignore = false;
    startFetching();
    return () => {
      ignore = true;
    }
  }, [person]);
  ...
}

```

다만 Effect에서 직접 데이터 패칭을 하는 것은 추후 캐싱/서버 렌더링 같은 최적화를 추가하기 어려워진다. 따라서 커스텀 훅을 사용하거나 적절한 패키지를 사용하는 것이 좋다.


### Displaying different content on the server and the client

서버 렌더링을 사용하는 경우, 서버에서 생성한 HTML과 클라이언트의 첫 렌더링 결과가 같아야 hydration이 정상적으로 이루어진다. 하지만 경우에 따라 클라이언트에서 다른 결과물을 나타내냐하는 경우가 있다. (localstorage 사용 등)
이럴때 다음과 같이 useEffect를 사용할 수 있다.
```js
function MyComponent() {  
	const [didMount, setDidMount] = useState(false);  
	
	useEffect(() => {  
		setDidMount(true);  
	}, []); 
	
	if (didMount) {  
		// ... return client-only JSX ...  
	} else {  
		// ... return initial JSX ...  
	}  
}
```

useEffect는 서버에서 실행되지않는다. 따라서 서버에서 생성한 HTML과 리액트의 첫 렌더링 결과는 같다. 따라서 hydration은 정상적으로 완료되고, Effect가 실행되면서 리렌더링이 이루어진다.

하지만 이러한 패턴은 되도록이면 피하라. 느린 환경에서는 컴포넌트 모양이 갑자기 바뀌기 때문이다. 대부분은 CSS를 사용하여 조건부로 다른 것을 표시할 수 있다.


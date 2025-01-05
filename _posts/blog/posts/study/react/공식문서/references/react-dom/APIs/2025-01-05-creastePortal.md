https://react-ko.dev/reference/react-dom/createPortal
```jsx
<div>  
	<SomeComponent />  
	{createPortal(children, domNode, key?)}  
</div>
```

### parameters
- `children` : JSX or 문자열 or 숫자 등 리액트로 렌더링 할 수 있는 모든 것
- `domNode` : `document.getElementById()`가 반환하는 것과 같은 일부 DOM 노드
- `key` (optional) : 포털의 키로 사용할 고유 문자열 또는 숫자


### Caveats
- 포털의 이벤트는 DOM 트리가 아닌 React 트리에따라 전파된다
- 포털은 DOM 노드의 물리적 배치만 변경한다. 다른 모든 면에서는 JSX는 이를 렌더링하는 React 컴포넌트의 자식 노드 역할을 한다.


## Usage
### Rendering React components into non-React server markup
리액트 루트가 리액트로 빌드되지 않은 정적페이지 또는 서버렌더링 페이지의 일부일 때 유용할 수 있다.
여러 개 개별 React 루트를 사용하는 것보다 포털을 사용하여 앱의 일부가 DOM의 다른 부분에 렌더링 되더라도 앱을 공유 state를 가진 단일 React 트리로 처리할 수 있다.

### Rendering React components into non-React DOM nodes
포털을 사용해 React 외부에서 관리되는 DOM 노드의 콘텐츠를 관리할 수도 있다

```jsx
import { useRef, useEffect, useState } from 'react';
import { createPortal } from 'react-dom';
import { createMapWidget, addPopupToMapWidget } from './map-widget.js';

export default function Map() {
  const containerRef = useRef(null);
  const mapRef = useRef(null);
  const [popupContainer, setPopupContainer] = useState(null);

  useEffect(() => {
    if (mapRef.current === null) {
      const map = createMapWidget(containerRef.current);
      mapRef.current = map;
      const popupDiv = addPopupToMapWidget(map);
      setPopupContainer(popupDiv);
    }
  }, []);

  return (
    <div style={{ width: 250, height: 250 }} ref={containerRef}>
      {popupContainer !== null && createPortal(
        <p>Hello from React!</p>,
        popupContainer
      )}
    </div>
  );
}

```

```jsx
import 'leaflet/dist/leaflet.css';
import * as L from 'leaflet';

export function createMapWidget(containerDomNode) {
  const map = L.map(containerDomNode);
  map.setView([0, 0], 0);
  L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
    maxZoom: 19,
    attribution: '© OpenStreetMap'
  }).addTo(map);
  return map;
}

export function addPopupToMapWidget(map) {
  const popupDiv = document.createElement('div');
  L.popup()
    .setLatLng([0, 0])
    .setContent(popupDiv)
    .openOn(map);
  return popupDiv;
}

```

- 리액트 외부에 있는 div의 내용을 리액트를 사용하여 채워넣을 수 있음
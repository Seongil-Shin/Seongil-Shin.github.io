https://react-ko.dev/reference/react/useDeferredValue
UI 일부의 업데이트를 지연시킬 수 있는 React 훅

```js
const deferredValue = useDeferredValue(value)
```

### Parameter
- `value` : 지연시키려는 값. 어떤 타입이든 될 수 있다

### Returns
- `value` : 지연된 값
	- 초기 렌더링 중에는 사용자가 제공한 값과 동일함
	- 업데이트 발생 시 React는 먼저 이전 값으로 리렌더링을 시도하고, 백그라운드에서 다시 새 값으로 리렌더링을 시도 

### Caveats
- 렌더링 중에 새 객체를 생성하고 바로 `useDeferredValue`에 전달하면 매 리렌더링마다 값이 달라져 매번 백그라운드 리렌더링이 발생한다.
- `useDeferredValue`는 백그라운드에서 리렌더링 중, 또 다른 업데이트를 받으면 그 백그라운드 리렌더링을 중단할 수 있다.
	- ex) 리렌더링 되는 시간보다 사용자 입력이 더 빠르면, 사용자 입력이 모두 종료된 후에 업데이트 됨
- `Suspense`와 통합된다. 새값으로 인한 백그라운드 업데이트로 인해 UI가 일시 중단돼도 사용자에게 풀백이 표시 되지 않는다. 백그라운드 리렌더링이 완료될 때까지 기존의 값이 표시된다.
- `useDeferredValue`는 그 자체로 추가 네트워크 요청을 막지 않는다.
- 리액트는 원래의 리렌더링을 완효하자마자, 즉시 새로운 지연된 값으로 백그라운드 리렌더링을 시작한다. 그러나 이벤트로 인한 업데이트(타이핑 등)는 백그라운드 리렌더링을 중단하고 우선순위를 갖는다.
- `useDeferredValue`로 인한 백그라운드 리렌더링은 화면에 커밋될 때까지 Effect를 실행하지 않는다. 백그라운드 리렌더링이 중단 되면 데이터가 로드되고 UI가 업데이트된 후에 해당 Effect가 실행된다.

## Usage

### Showing stale content while fresh content is loading
`useDeferredValue`는 지연되는 동안 이전 화면을 노출하며, Suspense 폴백도 띄우지 않는다. 따라서 오래 걸리는 걸 리렌더링할 때 이전 값으로 화면을 띄우기 위해 사용할 수 있다.


### Indicating that the content is stale
UI가 지연되는 동안 현재 화면이 오래된 내용이라는 것을 보여줄 수도 있다.
```js
export default function App() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  const isStale = query !== deferredQuery;
  return (
    <>
      <label>
        Search albums:
        <input value={query} onChange={e => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>Loading...</h2>}>
        <div style={{
          opacity: isStale ? 0.5 : 1,
          transition: isStale ? 'opacity 0.2s 0.2s linear' : 'opacity 0s 0s linear'
        }}>
          <SearchResults query={deferredQuery} />
        </div>
      </Suspense>
    </>
  );
}
```


### Deferring re-rendering for a part of the UI
무거운 컴포넌트를 리렌더링 시킬 때, 그 컴포넌트가 리렌더링하는 동안 나머지 UI를 차단하지 않도록 최적화를 위해 사용할 수 있다.

이때 그 무거운 컴포넌트를 `memo`로 감싸야한다. 그렇지 않으면 어쨌든 부모 컴포넌트가 업데이트되면 다시 렌더링해야하므로 최적화의 취지가 무색해진다.
```js
import { useState, memo } from 'react';
import SlowList from './SlowList.js';

export default function App() {
  const [text, setText] = useState('');
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <SlowList text={text} />
    </>
  );
}


const SlowList = memo(function SlowList({ text }) {
  // Log once. The actual slowdown is inside SlowItem.
  console.log('[ARTIFICIALLY SLOW] Rendering 250 <SlowItem />');

  let items = [];
  for (let i = 0; i < 250; i++) {
    items.push(<SlowItem key={i} text={text} />);
  }
  return (
    <ul className="items">
      {items}
    </ul>
  );
});
```


- 구 가이드 : https://ko.legacy.reactjs.org/docs/forwarding-refs.html
- 신 가이드 : https://react.dev/learn/manipulating-the-dom-with-refs


### 요약  
React는 렌더 출력과 DOM을 자동으로 일치시켜주므로 컴포넌트가 DOM을 직접 조작할 필요가 없습니다. 그러나 때로는 React가 관리하는 DOM 요소에 액세스해야 할 때가 있습니다. 이를 위해 ref를 사용해야 합니다.  
  
### 사실  
- 🏛️ React 컴포넌트가 관리하는 DOM 노드에 액세스하려면 `ref` 속성을 사용해야 합니다.  
- 🎓 `ref` JSX 속성과 `useRef` Hook의 관계에 대해서 알아야 합니다.  
- 🌐 다른 컴포넌트의 DOM 노드에 액세스하는 방법을 이해해야 합니다.  
- 🧠 React에서 관리하는 DOM을 수정하는 경우에 대한 안전한 사용 사례도 알아야 합니다.  
  
### 배울 내용  
- `ref`를 사용하여 React가 관리하는 DOM 노드에 접근하는 방법  
- `ref` JSX 속성과 `useRef` Hook의 관계  
- 다른 컴포넌트의 DOM 노드에 접근하는 방법  
- React가 관리하는 DOM을 수정해도 안전한 경우  
  
### DOM 노드에 접근하기  
1. React가 관리하는 DOM 노드에 접근하려면 `useRef` Hook을 먼저 가져와야 합니다.  
2. 컴포넌트 내에서 `useRef`를 사용하여 ref를 선언합니다.  
3. 접근하려는 JSX 태그의 `ref` 속성에 해당 ref를 전달합니다.  
4. `useRef` Hook는 `current`라는 단일 속성을 가진 객체를 반환합니다. 초기에는 `ref.current`가 `null`일 것입니다. React가 DOM 노드를 생성하면 이 `<div>`에 대한 참조가 `myRef.current`에 저장됩니다.  

### DOM 수정이 필요한 경우의 주의사항  
- 대부분의 경우, DOM 수정은 `ref`를 사용하여 해결할 수 있습니다. 그러나 React가 관리하는 DOM을 수정하는 경우 코드가 취약해질 수 있습니다.  
- 예제에서는 React가 관리하지 않는 DOM을 강제로 수정하면 문제가 발생하는 상황을 보여줍니다.  
- DOM을 수정하는 경우 React의 변경과 충돌할 수 있으므로 주의해야 합니다.  

### (\*) ref list 만들기
- https://react.dev/learn/manipulating-the-dom-with-refs#how-to-manage-a-list-of-refs-using-a-ref-callback
- `ref` 속성에 함수를 넣는 [ref callback](https://react.dev/reference/react-dom/components/common#ref-callback)을 사용할 수 있다. ref를 set 할 때는 DOM node로 호출하며, 컴포넌트가 사라질때는 null로 호출 된다.  (ref는 어떤 값이든 가질 수 있다.)
```js
export default function CatFriends() {
  const itemsRef = useRef(null); 

  function scrollToId(itemId) {
    const map = getMap();
    const node = map.get(itemId);
    node.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  function getMap() {
    if (!itemsRef.current) {
      // Initialize the Map on first usage.
      itemsRef.current = new Map(); // Refs can hold any values!
    }
    return itemsRef.current;
  }

  return (
      <div>
        <ul>
          {catList.map(cat => (
            <li
              key={cat.id}
              ref={(node) => {
                const map = getMap();
                if (node) {
                  map.set(cat.id, node);
                } else {
                  map.delete(cat.id);
                }
              }}
            >
              <img
                src={cat.imageUrl}
                alt={'Cat #' + cat.id}
              />
            </li>
          ))}
        </ul>
      </div>
  );
}
```
  
### 다른 컴포넌트의 DOM 노드 접근하기  
- 내장된 `<input />`과 같은 브라우저 요소에 `ref`를 추가하면 React는 해당 `ref`의 `current` 속성을 해당 DOM 노드에 설정합니다.  
- 하지만 `<MyInput />`과 같은 사용자 정의 컴포넌트에는 기본적으로 `null`이 설정됩니다.  
- 다른 컴포넌트의 DOM 노드에 직접 접근하는 것은 취약한 코드를 만들 수 있으므로 조심해야 합니다.  
- 컴포넌트가 자체 DOM 노드를 노출하려면 `forwardRef`를 사용하여 전달(ref forwarding)할 수 있습니다.  
  
### (\*)DOM 노드에 접근하는 시점  
- 렌더링 중에는 일반적으로 `ref`에 접근하지 않아야 합니다. 첫 번째 렌더링에서는 DOM 노드가 아직 생성되지 않았으므로 `ref.current`는 `null`일 것입니다.  
- React는 커밋 중에 `ref.current` 값을 설정합니다. DOM 업데이트 전에 영향을 받는 `ref.current` 값을 `null`로 설정한 다음, DOM 업데이트 후에 즉시 해당 DOM 노드로 설정합니다.  
- 보통 ref는 이벤트핸들러에서 필요하다. 만약 이벤트에서 사용하는게 아니라면, effect가 필요한 걸수도 있다.
  
### (\*)`flushSync`를 사용한 상태 업데이트 동기화  
- https://react.dev/learn/manipulating-the-dom-with-refs#flushing-state-updates-synchronously-with-flush-sync
- 상태 업데이트는 보통 큐에 쌓이고 비동기적으로 처리됩니다. 이로 인해 상태 업데이트가 DOM을 즉시 업데이트하지 않는 문제가 발생할 수 있습니다.  
- 이 문제를 해결하기 위해 `flushSync`를 사용하여 상태 업데이트를 동기적으로 처리할 수 있습니다. 이렇게 하면 업데이트 후 즉시 DOM이 업데이트되므로 문제를 해결할 수 있습니다.  
  
### `ref`를 사용한 DOM 조작의 최적 관행  
- `ref`는 React에서 DOM을 우회하는 수단입니다. 주로 focus, scroll position, 브라우저 API를 호출하는 경우에 사용합니다.  
- DOM을 변경하는 경우 React가 수행하는 변경과 충돌할 수 있으므로 주의해야 합니다.  
- focus()나 scroll과 같은 비파괴적인(DOM을 변경하지 않는) 작업을 수행할 때는 아무런 문제가 없음
- React가 관리하지 않는 DOM을 ref로 수정하는 경우에는 큰 문제가 없다(You can safely modify parts of the DOM that React has _no reason_ to update.)
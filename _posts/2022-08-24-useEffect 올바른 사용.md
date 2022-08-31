---
title : useEffect 올바른 사용
author : 신성일
date : 2022-08-24 14:00:00 +0900
categories : [study, react]
tags: [react]
---



# **useEffect 올바른 사용**

## **props, state 변경에 따라 다른 데이터, state를 업데이트해야할 때**

**useEffect를 사용하면 안된다.**

props, state가 변경이 되면 리렌더링이 발생하는데, 리렌더링이 끝난 후 useEffect가 호출되어 다시 리렌더링이 발생하기 때문이다.

따라서 다음과 같이 코드를 변경해야한다.

<br/>

### **데이터를 업데이트해야하는 경우**

```react
// useEffect로 업데이트
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');

  // Avoid: redundant state and unnecessary Effect
  const [visibleTodos, setVisibleTodos] = useState([]);
  useEffect(() => {
    setVisibleTodos(getFilteredTodos(todos, filter));
  }, [todos, filter]);

  // ...
}
```

```react
// useEffect 사용하지 않음 
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  // This is fine if getFilteredTodos() is not slow.
  const visibleTodos = getFilteredTodos(todos, filter);
  // ...
}
```

컴포넌트가 리렌더링할때마다 컴포넌트의 코드가 위에서부터 다시한번씩 실행되며 `getFilteredTodos()`도 다시 호출되기 때문에, 최신값으로 업데이트 된다.

만약 `getFilteredTodos()`가 비싼 연산이라면 `useMemo()`를 통해 최적화를 하자

```react
import { useMemo, useState } from 'react';

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  // Does not re-run getFilteredTodos() unless todos or filter change
  const visibleTodos = useMemo(() => getFilteredTodos(todos, filter), [todos, filter]);
  // ...
}
```

<br/>

### **props 변경에 따라 상태를 초기화 해야 하는 경우**

유저 변경에 따라 comment 상태를 초기화해야하는 경우

```react
// useEffect 사용
export default function ProfilePage({ userId }) {
  const [comment, setComment] = useState('');

  // 🔴 Avoid: Resetting state on prop change in an Effect
  useEffect(() => {
    setComment('');
  }, [userId]);
  // ...
}
```

userId 변경 --> 리렌더링 --> comment 변경 --> 리렌더링. 총 2번 리렌더링이 발생한다.

```react
// 리팩토링
export default function ProfilePage({ userId }) {
  return (
    <Profile
      userId={userId} 
      key={userId}
    />
  );
}

function Profile({ userId }) {
  // ✅ This and any other state below will reset on key change automatically
  const [comment, setComment] = useState('');
  // ...
}
```

key값을 사용하여, userId가 변경될때마다 `Profile` 컴포넌트를 초기상태에서 가져오도록 한다.

<br/>

### props 변경에 따라 특정 상태만 업데이트 해야하는 경우

```react
// useEffect 사용
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // 🔴 Avoid: Adjusting state on prop change in an Effect
  useEffect(() => {
    setSelection(null);
  }, [items]);
  // ...
}
```

```react
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // Better: Adjust the state while rendering
  const [prevItems, setPrevItems] = useState(items);
  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null);
  }
  // ...
}
```

items의 이전상태를 담는 상태를 생성하여 리렌더링을 하나로 줄였음.

<br/>

## **useEffect vs EventHandler**

쇼핑몰에서 카트에 물건을 넣었을 때 모달을 통해 정상적으로 물품을 담았는지 유저에게 노티피케이션을 알려줘야한다.

다음은 useEffect를 사용한 예시다. product가 카트에 추가되면, `isInCart` 상태가 `true`가 되어 useEffect에 등록한 콜백이 호출되고, Notification이 뜨는 방식이다. 

모달을 보여줘야하는 버튼이 여러개이면 일일히 각 핸들러에 `showNotification` 콜백을 붙이는 것보다 useEffect에 넣는게 바람직하다고 착각할 수 있다.

```react
function ProductPage({ product, addToCart }) {
  // 🔴 Avoid: Event-specific logic inside an Effect
  useEffect(() => {
    if (product.isInCart) {
      showNotification(`Added ${product.name} to the shopping cart!`);
    }
  }, [product]);

  function handleBuyClick() {
    addToCart(product);
  }

  function handleCheckoutClick() {
    addToCart(product);
    navigateTo('/checkout');
  }
  // ...
}
```

하지만 아니다.

페이지 전환 시 전역상태 값인 `product.isInCart`가 `true`면 예상하지 못한 버그가 생길 수 있기 때문이다.

또 유저는 상태가 변경되었을 때가 아닌, 버튼을 눌렀을 때 Notification이 뜨기를 기대할 것이다.

우리의 코드가 상태를 변경했을 때 작동하는 것이 의도인지, 유저의 이벤트에 따라 UI를 변경하는 것이 주 목적인지 생각해볼때, 버그를 양산할 수 있는 useEffect보다 이벤트 핸들러 내부에서 Notification을 호출하는 것이 바람직하다.

> useEffect, eventHandler 둘 중 어디에 코드를 작성해야할지 모르겠다면, 코드가 언제 실행되어야하는지 생각해봐라. useEffect는 컴포넌트가 렌더링 되는 시점에 유저에게 표현되어야할 로직을 실행할 때 사용한다.

<br/>

## **data fetching**

> data fetching은 해도 된다. 하지만 race condition을 고려해야한다.

리액트18 strict mode에서 컴포넌트 렌더링 시 useEffect 내부에서 ajax가 일어난다면, 렌더 -> fetch -> 리렌더 - refetch가 일어난다.

이는 api 서버 비용이 두 배로 든다는 점만 감안하면 괜찮다. 사용자는 fetch/refetch간 동일한 데이터가 응답된다면 UI에 변경점을 느끼지 못하기 때문이다.

하지만 **특정 상태나 Props 변경에 따라 뷰가 바뀌는 경우**라면 어떨까?

### **언마운트에서의 setState**

다음의 코드는 userId가 변경될 때마다 `startFetching`이 호출된다.

```react
useEffect(() => {
  let ignore = false;

  async function startFetching() {
    const json = await fetchTodos(userId);
    if (!ignore) {
      setTodos(json);
    }
  }

  startFetching();

  return () => {
    ignore = true;
  };
}, [userId]);
```

컴포넌트가 언마운트 되었을 때는 ignore를 true로 만들어, 클로져를 사용하는 `startFetching`내부에서 `setTodos`는 언마운트 이후에도 호출되지 않는다.

하지만 이러한 로직을 **매번 따로 작성**해야하는 문제점이 남아있다.

### **race condition**

한 페이지에서 특정 id값을 기준으로 모달같은 공통 ui를 그려주는 경우에는 위의 로직이 도움이 되지 않는다.

다음처럼 sartFetching이 테이블 컴포넌트에 있는 open modal을 누를때마다 호출된다고 가정해보자.

![img](https://velog.velcdn.com/images/jay/post/1719e0bf-4c68-433a-9b5e-f0b8e7c4103f/image.png)

`id 3의 open modal --> close modal --> id 4의 open modal` 과 같은 시나리오에서, id 4가 먼저 응답이 올지, id 3가 먼저 응답이 올지 알수가 없다.

이러한 문제를 `race condition`이라고 한다. useEffect 에서 setState를 호출하는 것만으로는 이러한 상황을 방지할 순 없다. 특별한 조치를 취해야 막을 수 있다. (서버에서 온 id와 props으로 온 id가 일치하는지 확인, AbortController 등)

### **특정 상태가 변경되었을 때 데이터를 호출하는 상황은?**

서버자원이 동나지 않도록 조심해야한다.

어떤 버그상황에도 대비할 수 있게 useEffect를 써야하고, dependencies의 상태 변경이 예측할 수 있는 속도로 변경되는 것을 보장할 수 있어야한다.

하지만 어려운 일이다.

따라서 동일한 api로 호출하는 경우 이전에 받아온 데이터를 재사용하는 방법으로 해결할 수 있다. 바로 `캐시`를 이용하는 방법이다. 이 캐시를 이용하기 위해 `react-query`, `swr` 등 data fetch 라이브러리를 사용할 수 있다.





<br/>

<br/>

## **출처**

https://velog.io/@jay/you-might-need-useEffect-diet
---
title: React에서 Key를 사용하는 이유
author: 신성일
date: 2022-07-28 22:00:00 +0900
categories: [study, react]
tags: [react]
---

리액트로 개발을 하다가 리스트를 만들면, 다음과 같이 key를 사용하라는 안내문이 뜬다.

![key_error](/assets/img/2022-07-28-react-key/key_error.png)

평소엔 무시하고 key를 그냥 넣었지만, 왜 그런지 갑자기 궁금해져 찾아보았다.

## Recursing On Children

해당 내용은 리액트 공식문서에 [Recursing On Children](https://reactjs.org/docs/reconciliation.html#recursing-on-children)으로 나와있다.

리액트 리렌더링 과정에서 리스트를 처리할때, 리액트는 기본적으로 실제 DOM의 리스트와 virtual DOM의 변경된 리스트를 비교한다.

```html
// 전
<ul>
   <li>first</li>
   <li>second</li>
</ul>

// 후
<ul>
   <li>first</li>
   <li>second</li>
   <li>third</li>
</ul>
```

예를 들어, 위와 같이 기존 리스트의 마지막에 새로운 요소가 추가되었다고 해보자. 이때 리액트는 [비교알고리즘](https://ko.reactjs.org/docs/reconciliation.html#the-diffing-algorithm)을 통해 리스트를 차례대로 비교한다. 먼저 `<li>first</li>`이 서로 같다는 걸 확인하고, `<li>second</li>`이 같다는 걸 확인한다. 그리고 `<li>third</li>`이 다르다는 걸 확인하고, `<li>third</li>` 을 트리에 추가하는 리렌더링을 수행한다.

이와 같이 맨 마지막에 요소를 추가하는 경우, 변경된 요소만 추가하기에 문제 없다. 하지만 문제는 다음과 같은 경우이다.

```html
<ul>
   <li>Duke</li>
   <li>Villanova</li>
</ul>

<ul>
   <li>Connecticut</li>
   <li>Duke</li>
   <li>Villanova</li>
</ul>
```

위와 같이 요소를 맨 위에 추가했을 경우 문제가 된다. 첫번째 예시처럼 비교를 했을때, 세 `<li>` 모두 다르기에, 리액트는 세 자식을 모두 리렌더링 시킨다.

## Keys

Recursing on children 에서 봤던 문제를 해결하기 위해 리액트는 `key` 속성을 지원한다. 자식들이 key를 가지고 있다면, 리액트는 key를 통해 기존 트리와 이후 트리의 자식들이 일치하는지 확인한다.

```html
<ul>
   <li key="2015">Duke</li>
   <li key="2016">Villanova</li>
</ul>

<ul>
   <li key="2014">Connecticut</li>
   <li key="2015">Duke</li>
   <li key="2016">Villanova</li>
</ul>
```

이렇게 키를 추가한다면, 리액트는 키가 같은 자식은 이전과 같은 자식이라고 인지하고 리렌더링하지 않는다. 오직 키가 변경된 요소만 리렌더링 하여 매우 효율적이 된다.

## 주의점

key는 형제 사이에서 `유일`해야한다. 즉, 같은 리스트 안에서 key는 유일해야 한다는 말이다.

key를 추가하라는 안내문이 뜬다는 이유로 나를 포함한 많은 이들이 다음과 같은 실수를 저지른다.

```react
{state.list.map((todo, index) => (
    <ToDo key={index} {...todo} />
))}
```

이처럼 인덱스를 key로 사용하면, 만약 새로운 요소를 리스트의 맨 앞에 추가했을 경우 리스트의 key는 새로 추가된 요소부터 0 ~ n 의 값을 가진다.

```html
<ul>
   <li key="0">Duke</li>
   <li key="1">Villanova</li>
</ul>

<ul>
   <li key="0">Connecticut</li>
   <li key="1">Duke</li>
   <li key="2">Villanova</li>
</ul>
```

리액트는 key를 통해 리렌더 해야 할 요소를 뽑아낸다. 그런데 이 예시에서는 새로 맨 앞에 추가된 요소가 이전 리스트에 존재했던 `key="0"`을 갖는다. 이로 인해 리액트는 새로 추가된 요소를 리렌더링 하지 않고, 오히려 기존에 있던 맨 마지막 요소를 리렌더링 한다.(기존 리스트에선 2라는 key는 없었기에)

이와 같이 각 index를 key로 사용하면 의도하지 않는 방식으로 동작할 수 있다. [예시](https://ko.reactjs.org/redirect-to-codepen/reconciliation/index-used-as-key) [예시 해결](https://ko.reactjs.org/redirect-to-codepen/reconciliation/no-index-used-as-key)

리스트에 `key`를 지정해주지 않으면 리액트에서는 기본적으로 index를 `key`로 사용한다. 따라서 특별한 경우가 아니라면 key는 유일한 값으로 지정해주자.

> 특별한 경우
>
> 1. 리스트가 변화되지 않을 경우
> 2. 성능에 신경쓸만큼 긴 리스트가 아닌데, 식별자를 생성하기 어려운 경우

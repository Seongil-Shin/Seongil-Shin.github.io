---
title : virtual DOM와 리렌더링
author : 신성일
date : 2022-07-17 14:00:00 +0900
categories : [study, react]
tags: [react, virtual dom]
---



Virtual DOM은 DOM을 추상화한 가상의 객체이다.

<br/>

## DOM 이란?

HTML 문서를 파싱하여 문서의 구성요소들을 객체로 구조화하여 나타낸 것.

![img](/assets/img/2022-07-17-virtual Dom/img.png)

HTML Elements, 속성, CSS style, Events, Methods 를 구조화해서 나타낸 객체고, 이 객체를 이용해서 웹페이지 구성요소를 제어할 수 있다.



**Virtual DOM**은 이러한 DOM을 추상화한 가상의 객체이다.

<br/>

## Virtual DOM 이 해결하고자 하는 문제

React 작동 방식은 상태가 변경된 컴포넌트는 그 컴포넌트부터 자식 컴포넌트까지 리렌더링한다. 하지만 이런 식의 교체는 많은 PC자원을 소모하게 된다. 이는 브라우저의 렌더링 방식 때문이다. 

1. HTML을 파싱하여 DOM 객체를 생성하고, CSS를 파싱하여 스타일 규칙을 만든다.
2. 이 두개를 합쳐 실제로 웹 브라우저에 보여질 요소를 표현한 "렌더트리"를 만든다.
3. 이 렌더 트리를 기준으로 레이아웃을 배치하고, 색을 칠하는 등의 작업을 한다.

만약 변경해야할 사항이 100개라면, 100번 위와 같은 과정을 반복하게 된다.

<br/>

## 어떻게 해결했는가?

Virtual DOM이라는 DOM을 추상화한 가상의 객체를 메모리에 만들어 놓고, DOM과 유사한 역할을 담당하게 한다. 

만약 변경사항이 발생할 경우, DOM을 직접 수정하는 것이 아니라 중간단계로 Virtual DOM을 수정하고 Virtual DOM을 통해서 DOM을 수정하게 한다. 이를 통해 Virtual DOM에 변경내역을 한 번에 모아서(버퍼링) 실제 DOM과 변경된 Virtual DOM의 차이를 판단한 후, 구성요소의 변경이 필요한 부분만 찾아 그에 따른 렌더링을 한번만 하도록 하여 해결하였다.

즉, 리액트에서 상태변화가 발생했을 경우 교체해야할 컴포넌트를 먼저 Virtual DOM에서 교체한다음 실제 DOM과 비교하여 rerender 해야할 부분만 rerender한다는 뜻이다.

<br/>

## 리렌더링 과정

1. 상태 변경, props 변경 등으로 컴포넌트 리렌더링 시작
2. 구현부 실행, hook 실행, 내부 변수 및 함수 재생성
3. return 실행, 렌더링 시작
4. 렌더단계 : 새로운 가상 DOM 생성 후 이전 가상 DOM과 비교해 달라진 부분을 탐색하여 실제 DOM에 반영할 부분을 결정
5. 커밋단계 : 달라진 부분만 실제 DOM에 반영
6. useLayoutEffect : 브라우저가 화면에 Paint 하기 전에 useLayoutEffect에 등록해둔 effect 가 동기적으로 실행되며, 이때 상태가 변한다면 다시 리렌더링
7. Paint : 브라우저가 실제 DOM을 화면에 그림. componentDidUpdate 실행
8. useEffect : update되어 화면에 그려진 직후, useEffect에 등록해둔 effect가 비동기로 실행

<br/>

## virtual dom의 한계

- 0.1초마다 변경되는 화면을, Virtual DOM에 0.5초씩 모아서 렌더링할 수 있을까? 
  - 안된다. 동시에 변경되는 것에 한해서만 렌더링됨
- Virtual DOM은 무조건 빠른가?
  - 아니다. 반복렌더링 하지 않도록 해줘야한다. 
- Virtual DOM은 메모리에 있기에 메모리의 사용이 늘어난다.
- Virtual DOM을 조작하는 것도 엄청나게 많은 컴포넌트를 조작하게 된다면 오버헤드가 생기기마련이다. Virtual DOM 제어가 직접 DOM제어에 비해 상대적으로 비용이 적게 들 뿐이다. 

<br/>

## 오해

- React는 virtual DOM을 사용하기에 DOM API로 직접 엘리먼트를 수정하는 것보다 빠르다?
  - 아니다. React 도 결국 DOM API로 엘리먼트를 리렌더하기에 DOM API로 직접 수정하는 것보다 빠르지 않다.
  - 다만, 100번의 DOM API 사용을, Virtual DOM으로 모아 1번의 DOM API 사용으로 줄여주기에, virtual DOM 없이 리액트를 사용하는 것보다 더 빠른 것이다. 
  - 즉, Virtual DOM은 리액트를 빠르게 하는 수단이다.

<br/>

## 재조정(Reconciliation)

리액트에서 가상돔을 실제 DOM에 반영하는 과정을 `재조정`이라고 한다. 위 리렌더링 과정의 렌더에 해당한다. 이때 하나의 트리를 다른 트리로 변환하기 위한 최소한의 연산 수는 노드가 n일때 O(n^3)이다.

이는 매우 느리기에 리액트에서는 실제 DOM과 가상 DOM을 비교하기 위해 `diffing 알고리즘`을 사용한다. 리액트는 아래의 2가지 가정을 기반으로 O(n) 시간 복잡도를 가지는 휴리스틱 알고리즘을 구현하였다.

- 서로 다른 타입을 가진 두 엘리먼트는 다른 트리를 만들어낸다.
- 개발자가 key prop을 통해 자식 엘리먼트의 변경 여부를 표시할 수 있다.

좀 더 자세히, 리액트에서 두 개의 트리를 비교할 때 두 엘리먼트의 루트 엘리먼트부터 비교한다. 이후의 동작은 루트 엘리먼트의 타입에 따라 달라진다.

- 엘리먼트의 타입이 다른 경우
  - 두 루트 엘리먼트의 타입이 다르면, React는 이전 트리를 버리고 새로운 트리를 구축한다. `<a>` 에서 `<img>`로, `<Article>`에서 `<Comment>`로 바뀌는 경우를 말한다.
  - 트리를 버릴 때 이전 DOM 노드들은 모두 파괴된다. 컴포넌트 인스턴스는 `componentWillUnmount()`를 실행한다. 새로운 트리가 만들어질 떄 새로운 DOM 노드들이 DOM에 삽입된다. 그에 따라 컴포넌트 인스턴스는 `UNSAFE_componentWillMount()`가 실행되고 `componentDidMount()`가 실행된다. 이전 트리와 연관된 모든 state는 사라진다.
- DOM 엘리먼트의 타입이 같은 경우
  - 같은 타입의 두 React DOM 엘리먼트를 비교할 때, React는 동일한 속성은 유지하고 변경된 속성들만 갱신한다. 
  - style이 갱신될 때는 변경된 속성만 변경한다. `style={{color:"red", fontWeight:"bold"}}` -> `style={{color:"blue", fontWeight:"bold"}}`일때, fontWeight는 건들지 않고, color만 수정한다.
- 같은 타입의 컴포넌트 엘리먼트
  - 컴포넌트가 갱신되면 인스턴스는 동일하게 유지되어 렌더링 간 state가 유지된다. React는 새로운 엘리먼트의 내용을 반영하기 위해 현재 컴포넌트 인스턴스의 props를 갱신한다. 이때 해당 인스턴스의 `componentDidUpdate`를 호출한다. 

이후 render() 메서드가 호출되고 diffing 알고리즘은 재귀적으로 자식에 대해 처리한다. 리액트는 기본적으로 동시에 두 리스트를 순회하고 차이점이 있으면 변경을 생성한다.

예를 들어 리스트 끝에 엘리먼트를 추가하면 두 트리 사이의 변경은 잘 작동한다.

```html
<ul>
  <li>first</li>
  <li>second</li>
</ul>

<ul>
  <li>first</li>
  <li>second</li>
  <li>third</li>
</ul>
```

하지만 리스트의 맨 앞에 엘리먼트를 추가하면 성능은 좋지 않다. 모든 엘리먼트를 다시 생성하기 때문이다.

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

이러한 문제를 해결하기 위해 리액트는 `key` 속성을 제공한다. 자식들이 같은 key를 가지고 있으면 React는 key를 통해 기존 트리와 이후 트리의 자식들이 일치하는지 확인한다.

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

리액트는 이제 `2014` key를 가진 엘리먼트가 추가되었고, 2015, 2016 key를 가진 엘리먼트는 그저 이동만 하면 되는 것을 알 수 있다. 이때 사용하는 key는 형제들 사이에서 유일하면 된다. 

최후의 경우 배열의 인덱스를 key로 사용할 수 있다. 하지만 이 경우 배열이 재배열되면 컴포넌트의 state가 꼬일 수 있다. 컴포넌트 인스턴스는 key를 기반으로 갱신되고 재사용되기 때문에, 인덱스를 key로 사용하면 항목의 순서가 바뀌었을 때 key 또한 바뀌기 때문이다. [예시](https://codepen.io/pen?&editors=0010&layout=left) [개선된 예시](https://codepen.io/pen?&editors=0010&layout=left)

<br/>

# 출처

https://jeong-pro.tistory.com/210

https://stackoverflow.com/questions/61245695/how-exactly-is-reacts-virtual-dom-faster

https://narup.tistory.com/272
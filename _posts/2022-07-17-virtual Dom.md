---
title : virtual DOM
author : 신성일
date : 2022-07-17 14:00:00 +0900
categories : [study, react]
tags: [react, virtual dom]
---



# Virtual DOM이란?

DOM을 추상화한 가상의 객체이다.



## DOM 이란?

HTML 문서를 파싱하여 문서의 구성요소들을 객체로 구조화하여 나타낸 것.

![img](/assets/img/2022-07-17-virtual Dom/img.png)

HTML Elements, 속성, CSS style, Events, Methods 를 구조화해서 나타낸 객체고, 이 객체를 이용해서 웹페이지 구성요소를 제어할 수 있다.



**Virtual DOM**은 이러한 DOM을 추상화한 가상의 객체이다.



## Virtual DOM 이 해결하고자 하는 문제

React 작동 방식은 상태가 변경된 컴포넌트는 그 컴포넌트부터 자식 컴포넌트까지 통채로 새 상태의 컴포넌트로 교체하는 방식으로 동작한다.

하지만 이런식의 교체는 많은 PC자원을 소모하게 된다. 이는 브라우저의 렌더링 방식 때문이다. 

1. HTML을 파싱하여 DOM 객체를 생성하고, CSS를 파싱하여 스타일 규칙을 만든다.
2. 이 두개를 합쳐 실제로 웹 브라우저에 보여질 요소를 표현한 "렌더트리"를 만든다.
3. 이 렌더 트리를 기준으로 레이아웃을 배치하고, 색을 칠하는 등의 작업을 한다.

만약 변경해야할 사항이 100개라면, 100번 위와 같은 과정을 반복하게 된다.



## 어떻게 해결했는가?

Virtual DOM이라는 DOM을 추상화한 가상의 객체를 메모리에 만들어 놓고, DOM과 유사한 역할을 담당하게 한다. 

만약 변경사항이 발생할 경우, DOM을 직접 수정하는 것이 아니라 중간단계로 Virtual DOM을 수정하고 Virtual DOM을 통해서 DOM을 수정하게 한다. 이를 통해 Virtual DOM에 변경내역을 한 번에 모아서(버퍼링) 실제 DOM과 변경된 Virtual DOM의 차이를 판단한 후, 구성요소의 변경이 필요한 부분만 찾아 그에 따른 렌더링을 한번만 하도록 하여 해결하였다.

즉, 리액트에서 상태변화가 발생했을 경우 교체해야할 컴포넌트를 먼저 Virtual DOM에서 교체한다음 실제 DOM과 비교하여 rerender 해야할 부분만 rerender한다는 뜻이다.



## 한계

- 0.1초마다 변경되는 화면을, Virtual DOM에 0.5초씩 모아서 렌더링할 수 있을까? 
  - 안된다. 동시에 변경되는 것에 한해서만 렌더링됨
- Virtual DOM은 무조건 빠른가?
  - 아니다. 반복렌더링 하지 않도록 해줘야한다. 
- Virtual DOM은 메모리에 있기에 메모리의 사용이 늘어난다.
- Virtual DOM을 조작하는 것도 엄청나게 많은 컴포넌트를 조작하게 된다면 오버헤드가 생기기마련이다. Virtual DOM 제어가 직접 DOM제어에 비해 상대적으로 비용이 적게 들 뿐이다. 



## 오해

- React는 virtual DOM을 사용하기에 DOM API로 직접 엘리먼트를 수정하는 것보다 빠르다?
  - 아니다. React 도 결국 DOM API로 엘리먼트를 리렌더하기에 DOM API로 직접 수정하는 것보다 빠르지 않다.
  - 다만, 100번의 DOM API 사용을, Virtual DOM으로 모아 1번의 DOM API 사용으로 줄여주기에, virtual DOM 없이 리액트를 사용하는 것보다 더 빠른 것이다. 
  - 즉, Virtual DOM은 리액트를 빠르게 하는 수단이다.



# 출처

https://jeong-pro.tistory.com/210

https://stackoverflow.com/questions/61245695/how-exactly-is-reacts-virtual-dom-faster
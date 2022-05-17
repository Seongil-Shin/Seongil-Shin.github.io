---
title : velog 메모
author : 신성일
date : 2022-04-24 23:00:00 +0900
categories : [study, react]
tags: []
---

## 리액트 리렌더링

https://velog.io/@eunbinn/when-does-react-render-your-component

- children으로 넘어온 컴포넌트는 사용되는 컴포넌트가 리렌더링되도, 리렌더링의 영향을 받지 않는다.
- 리렌더링은 현재 컴포넌트에서 변화된 것이 없어도, 자식으로 전파된다.
  - but 현재 컴포넌트에 useMemo를 사용하면, 변화가 없을시 자식으로도 전파되지 않음

- useMemo도 써야할 때를 알아야한다. (구조변경 등의 방법으로 리렌더링을 막을 수 있음)
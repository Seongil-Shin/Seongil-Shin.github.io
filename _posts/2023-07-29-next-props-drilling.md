---
title: next.js props drilling
author: 신성일
date: 2023-08-05 16:00:00 +0900
categories: [study, next.js]
tags: [next.js]
---

# Data sharing (Solving props drilling)

next.js 13.4 이상 App router 기준으로 작성하였습니다.

## props drilling 예시

next.js에서 주로 발생하는 props drilling 형태는 다음과 같습니다.

- 먼저 필요한 데이터를 서버컴포넌트에서 fetch 합니다. API 노출 방지, 최적화 등 다양한 이유로 서버 컴포넌트(서버 사이드)에서 API를 호출합니다.

```js
// Server Component A
import ServerComponentB from "./ServerComponentB"

export default async function ServerComponentA() {
  const initialCount = await fetch(...);
  
  return <>
     <h3>I got {initialCount}</h3>
     <ServerComponentB initialCount={initialCount} />
  </>
}
```

- 데이터를 fetch한 서버컴포넌트에서 하위 서버컴포넌트로 데이터를 넘겨줍니다. `ServerComponentB`에서는 initialCount가 필요없지만, 하위 컴포넌트에서 필요하기에 받고, 넘겨줍니다.

```js
// Server Component B
import ServerComponentC from "./ServerComponentC"

export default function ServerComponentB({ initialCount }) {
  return <ServerComponentC initialCount={initialCount}/>
}
```

- `ServerComponentC`에서는 `initialCount`가 필요합니다. 필요한 작업을 수행 후 하위 컴포넌트를 불러옵니다. 그리고 그 하위 컴포넌트에도 `initialCount`가 필요하기에 props로 받은 데이터를 넘겨줍니다.

```js
// Server Component C
import ClientComponent from "./ClientComponent"

export default function ServerComponentC({ initialCount }) {
  return (
    <div>
      <h3>initial count is : {initialCount}</h3>
    	<ClientComponent initialCount={initialCount}/>
  	</div>
  )
}
```

- 최종적으로 최하위 컴포넌트에 도달하였습니다.

```js
// Client Component
'use client'

import {useState} from "react"

export default function ClientComponent({initialCount}) {
  const [count, setCount] = useState(initialCount);
  
  return (
    <>
      {count}
      <button onClick={() => setCount(prev => prev + 1)}>+1</button>
      <button onClick={() => setCount(prev => prev - 1)}>-1</button>
    </>
  )
}
```

이처럼 상위 컴포넌트에 있는 데이터를 하위 컴포넌트에서도 가져야하고 이것을 props로 넘겨줄때, props drilling이 발생합니다. 중간에 `ServerComponentB`에서는 `initialCount`가 필요없었지만 하위 컴포넌트에서 필요하기에 받고 넘겨줘야했습니다.

만약 일반적인 React 앱처럼 클라이언트 컴포넌트로만 구성되어있다면 글로벌 상태관리 라이브러리(redux, recoil..)나, Context API를 사용할 수 있습니다. 하지만 서버컴포넌트에서는 상태를 사용할 수 없기에 위와 같은 방법은 사용하기 어렵습니다. 

여기서는 서버컴포넌트가 있는 구조에서 props drilling 문제 없이 데이터를 공유할 수 있는 방법을 작성하였습니다. 

 <br/>

## Server Component -> Server Component

next.js에서 기본으로 제공해주는 기능인 `Automatic fetch() Requst Deduping`을 사용할 수 있습니다.
[next.js 공식문서](https://nextjs.org/docs/app/building-your-application/data-fetching#automatic-fetch-request-deduping)에서의 설명에 따르면, 서버에서 `fetch`로 `GET`한 데이터는 rendering pass 동안 캐시된다고 합니다. 그리고 이 캐시는 server request 생명주기동안 유지되며 렌더링 과정이 끝날때까지 지속된다고 합니다.

따라서 서버컴포넌트끼리는 별도의 data sharing 기법이 필요하지 않고, 매번 필요한 데이터를 불러오면 됩니다. 

```js
// Server Component A
import ServerComponentB from "./ServerComponentB"

export default async function ServerComponentA() {
  const initialCount = await fetch(...);
  
  return <>
     <h3>I got {initialCount}</h3>
     <ServerComponentB/>
  </>
}
```

```js
// Server Component B
import ServerComponentC from "./ServerComponentC"

export default function ServerComponentB() {
  return <ServerComponentC/>
}
```

```js
// Server Component C
import ClientComponent from "./ClientComponent"

export default function ServerComponentC() {
  const initialCount = await fetch(...);
  return (
    <div>
      <h3>initial count is : {initialCount}</h3>
    	<ClientComponent initialCount={initialCount}/>
  	</div>
  )
}
```

이제 `ServerComponentB`에서는 `initialCount`를 받을 필요도, 넘겨줄 필요도 없습니다. `fetch()` 동안 결과가 캐시되어있기 때문에 두번이상 API 요청을 보내지 않습니다.

### 한계점

저는 개발을 하면서 서버컴포넌트에서 API 요청을 보낼때 주로 searchParams에 들어온 값을 기준으로 API를 호출했습니다.

```ts
// ex : 클라이언트 요청 주소 -> /notice?categoryId=4&searchValue=test
// categoryId=4, searchValue=test 로 데이터를 받아옴
export default async function NoticePage({ searchParams : {categoryId, searchValue } }) {
  const notices = await getNotices({ categoryId, searchValue })
  ...
}
```

하지만 next.js app router에서 `page.js`가 아닌 서버컴포넌트에서 `searchParams`를 얻어올 수 있는 공식적으로 제공하는 방법은 없습니다. 따라서 위에서 설명한, 데이터가 필요한 서버컴포넌트에서 api를 요청하는 건 어렵습니다. API 요청에 필요한 `searchParams`를 얻을 수 없기 때문입니다.

```js
// page.js가 아닌 어딘가의 서버컴포넌트
export default function NoticeList() {
  // error! categoryId, searvhValue를 알아낼 수 없음
  const notices = await getNotices({ categoryId, searchValue })
}
```

따라서 `Server Component -> Server Component data sharing`을 좀 더 잘 활용하려면, 하나의 Request가 살아있는 동안 여러 서버컴포넌트에서 그 Request의 searchParams에 접근가능해야합니다. 

공식적인 방법은 아니나 몇가지 생각해본/찾아본 방법은 다음과 같습니다.

1. 웹서버 활용

   next.js에서는 서버컴포넌트에 [headers()](https://nextjs.org/docs/app/api-reference/functions/headers)라는 함수를 제공합니다. 요청이 온 Request의 header을 제공해주는 함수입니다. 이 함수는 꼭 `page.js`만이 아닌 곳에서도 사용가능하기에, header에 searchParams가 들어있다면 `page.js`가 아닌 서버컴포넌트에서도 searchParams에 접근 가능합니다.

   ```nginx
   # nginx 예시
   server {
       location / {
           proxy_set_header X-Search-Params $args;
           proxy_pass http://localhost:3000;
       }
   }
   ```

   ```js
   import { headers } from 'next/headers'
   
   export default function NoticeList() {
     const header = headers() 
     const { categoryId, searchValue } = parseSearchParams(header.get("X-Search-Params"))
     
     const notices = await getNotices({ categoryId, searchValue })
   }
   ```

2. searchParams 정도는 props로 내려보내기

    `page.js`에서 searchParams가 필요한 서버컴포넌트까지 내려줘야하지만, `searchParams` 객체 하나만 내려주면 되기에 props drilling을 하나의 객체만 내려보내는 수준으로 유지할 수 있습니다.

   ```js
   export default function NoticeList({searchParams : {categoryId, searchValue }}) {
     const notices = await getNotices({ categoryId, searchValue })
   }
   ```

   관련해서 next.js 깃허브 discussions에서 논의된 [얘기](https://github.com/vercel/next.js/discussions/44005)가 있는데, searchParams에 대한 validation/parsing/error handling 없이 하위 컴포넌트로 searchParams를 넘겨줄 가능성은 적다는 댓글이 있었습니다. 가능성이 적은지는 모르겠으나, 확실히 파싱, 유효성 검증이 필요한 searchParams의 경우 `page.js`에서 먼저 받아 처리를 한 후, 하위 컴포넌트로 props로 넘겨주는게 더 좋은 방법으로 보입니다.

3. 서버단에서 data sharing 툴 사용

   꼭 searchParams에 국한되지 않더라도, 서버컴포넌트들 사이에서 데이터 or context를 공유하면 좋은 경우가 많습니다. (i18n, theme 등). [next.js discussions](https://github.com/vercel/next.js/discussions/42301)

   공식적으로 제공해주는 방법은 없지만, 서드파티 툴을 만드려는 시도는 존재합니다([server-only-context](https://www.npmjs.com/package/server-only-context), [react cache function](https://nextjs.org/docs/app/building-your-application/data-fetching/fetching-caching-and-revalidating#react-cache-function) 활용). 공식문서에서는 db와 같이 네이티브 JS 패턴을 사용하는 방법을 추천해주고 있습니다. [링크](https://nextjs.org/docs/getting-started/react-essentials#sharing-data-between-server-components)

<br/>

## Client Component -> Client Component

자주 있는 상황으로, 어떤 클라이언트 컴포넌트와 디렉터리 상 먼 곳에 있는 클라이언트 컴포넌트가 데이터 또는 상태를 공유해야하는 경우가 있을 수 있습니다.  이럴때는 일반적으로 리액트 앱을 개발할때 사용하는 전역 상태 관리 방법을 채택할 수 있습니다.  next.js 공식문서에서도 Context API 및 서드파티 라이브러리에 대한 설명에 있습니다. [링크](https://nextjs.org/docs/getting-started/react-essentials#using-context-in-client-components)

예시로는 Context API를 사용하였습니다.

```js
// layout.tsx
import { AppProvider } from '../lib/AppProvider';

export default function RootLayout({ children }) {
    return (
        <html lang="en">
            <body className={inter.className}>
                <AppProvider>{children}</AppProvider>
            </body>
        </html>
    );
}
```

```js
// lib/AppProvider.tsx
'use client';
import React, { createContext, useState, useContext } from 'react';

const AppContext = createContext();

export const AppProvider = ({ children }) => {
    const [state, setState] = useState(0);
    const value = {
        state,
        setState,
    };
    return <AppContext.Provider value={value}>{children}</AppContext.Provider>;
}
export const useAppContext = () => useContext(AppContext);
```

```js
// page.tsx
import Counter from '../components/counter';
import ServerComp from '../components/server';

export default function Home() {
    return (
        <main className={styles.main}>
            <ServerComp />
            <Counter />
        </main>
    );
}
```

```js
// components/counter.tsx
'use client';

import { useAppContext } from '../lib/AppProvider';

export default function Counter() {
    const { state, setState } = useAppContext();

    const increaseHandler = () => {
        setState(state + 1);
    };

    const decreaseHandler = () => {
        setState(state - 1);
    };
    // Use the state and setState as needed
    return (
        <>
            <div>{state}</div>
            <a href="#" onClick={increaseHandler}>
                increase
            </a>{' '}
            &nbsp;
            <a href="#" onClick={decreaseHandler}>
                decrease
            </a>
        </>
    );
}
```

- **오해** : Context Provider는 클라이언트 컴포넌트이고, 이것을 가장 상위에 두면 그 아래는 전부 클라이언트 번들에 포함될거라고 오해하는 경우가 있습니다. 그 이유는 next.js 공식문서에 아래와 같은 문구가 있기 때문인데요.

> Once `"use client"` is defined in a file, **all other modules imported into it,** **including child components**, are considered part of the client bundle
>
> https://nextjs.org/docs/getting-started/react-essentials#the-use-client-directive

클라이언트 컴포넌트의 자식 컴포넌트들은 클라이언트 번들로 간주된다는 내용입니다. 따라서 Context API Provider를 상위에 두면 그 하위 서버컴포넌트들도 모두 클라이언트 번들에 포함되어 서버컴포넌트를 사용하는 이유가 없어질것만 같습니다.

하지만 실험해본 결과, 클라이언트 번들에 포함되지 않았습니다. 이유는 ServerComponent가 ClientComponent(Context Provider)에 직접 import 되지 않고, props로 넘어갔기 때문입니다

next.js 공식 문서에 따르면 서버컴포넌트를 클라이언트 컴포넌트 밑에 두려면, import 하지 말고 prop(children)으로 넘겨줘야한다고 되어있습니다.  [링크](https://nextjs.org/docs/getting-started/react-essentials#nesting-server-components-inside-client-components)

```js
// Unsupported Pattern: Importing Server Components into Client Components

'use client'
 
// This pattern will **not** work!
// You cannot import a Server Component into a Client Component.
import ExampleServerComponent from './example-server-component'
 
export default function ExampleClientComponent({
  children,
}: {
  children: React.ReactNode
}) {
  const [count, setCount] = useState(0)
 
  return (
    <>
      <button onClick={() => setCount(count + 1)}>{count}</button>
      <ExampleServerComponent />
    </>
  )
}
```

```js
// Recommended Pattern: Passing Server Components to Client Components as Props

'use client'
 
import { useState } from 'react'
 
export default function ExampleClientComponent({
  children,
}: {
  children: React.ReactNode
}) {
  const [count, setCount] = useState(0)
 
  return (
    <>
      <button onClick={() => setCount(count + 1)}>{count}</button>

      {children}
    </>
  )
}
```

공식 문서에 설명되어 있는대로, import 대신 props(children)으로 넘겨주면 클라이언트 컴포넌트는 자식 컴포넌트를 위한 slot을 남기고, 서버컴포넌트 렌더가 종료되면 그 slot에 서버 컴포넌트가 렌더링 됩니다. 이 방식대로 한다면 서버컴포넌트는 클라이언트 번들에 포함되지 않고, 서버컴포넌트로써 동작합니다.

<br/>

## Server Component -> Client Component

"Server Component -> Server Component"에서 했던 것처럼, 한번 부른 fetch의 결과는 같은 요청이 살아있는 동안 캐시된다는 것을 활용할 수 있습니다. 데이터가 필요한 클라이언트 컴포넌트를 서버컴포넌트로 한번 감싸서 props drilling을 완화할 수 있습니다. [관련 링크(유튜브 영상, 소리주의)](https://www.youtube.com/watch?v=rbTzTXHkXA8&ab_channel=BogdanAdrian)

```js
// Server Component A
import ServerComponentB from "./ServerComponentB"

export default async function ServerComponentA() {
  const initialCount = await fetch(...);
  
  return <>
     <h3>I got {initialCount}</h3>
     <ServerComponentB />
  </>
}
```

```js
// Server Component B
import ServerComponentC from "./ServerComponentC"

export default function ServerComponentB() {
  return <ServerComponentC/>
}
```

```js
// Server Component C
import ClientComponent from "./ClientComponent"

export default function ServerComponentC() {
  const initialCount = await fetch(...);
  return 	<ClientComponent initialCount={initialCount}/>
}
```

```js
// Client Component
'use client'

import {useState} from "react"

export default function ClientComponent({initialCount}) {
  const [count, setCount] = useState(initialCount);
  
  return (
    <>
      {count}
      <button onClick={() => setCount(prev => prev + 1)}>+1</button>
      <button onClick={() => setCount(prev => prev - 1)}>-1</button>
    </>
  )
}
```

<br/>

## Client Component -> Server Component

props로 넘겨주는 방법이나, 다음과 같이 `cloneElement`를 이용한 방법이 있습니다.

```js
import React from "react";
import ClientComponent from "./ClientComponent"

export default function ParentComponent({ children }) {
  return (
    <ClientComponent>
       <ServerComponent/>
    </ClientComponent>
  ) 
}
```

```js
'use client'

import React from "react";

export default function ClientComponent({ children }) {
  return (
    <div>
       {React.cloneElement(children, {name:"test props"})}
    </div>
  ) 
}
```

```js
import React from "react";

export default function ServerComponent({ name }) {
  return (
    <div>
       {name}
    </div>
  ) 
}
```

하지만 가능한 것과는 별개로 크게 효용성이 있을지는 모르겠습니다. 이 방법은 클라이언트 컴포넌트의 바로 한단계 밑 서버컴포넌트로만 전달이 가능하기에, 클라이언트 컴포넌트에서 서버컴포넌트를 변경하려면 URL 쿼리를 변경하여 새로 페이지를 fetch하는게 나을 수도 있습니다. URL 쿼리를 변경하는 경우 한단계 밑 서버컴포넌트뿐만 아니라 관련있는 모든 서버컴포넌트들도 변경이 가능하기 떄문입니다.


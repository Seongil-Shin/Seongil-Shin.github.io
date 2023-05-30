---
title : (Typescript) React typescript hack
author : 신성일
date : 2022-01-22 01:00:00 +0900
categories : [study, react]
tags: []
---



React 에서 Typescript 사용시 참고할만한 Typing 기법을 기록한 곳





## React.HTMLArributes<[HTMLElement]>

한 Element에 부여될 속성 값들을 모두 참조할때 사용하면 좋다. 다음과 같이 기본 HTML 엘리먼트를 스타일링해서 사용할때 유용함

```typescript
export interface ITextButtonProps extends IProps, React.HTMLAttributes<HTMLDivElement> {
  text?: string | JSX.Element
}
```

위와 같이, `[]`로 감싸진 부분에 필요한 엘리먼트를 넣으면 된다.

하지만 위 방법이 통하지 않을 경우, 아래와 같이 엘리먼트 별로 구분된 속성을 사용하면 된다.

```typescript
const CustomInput = ({ error, ...props }: React.InputHTMLAttributes<HTMLInputElement> & IProps) => {
  return (
    <div>
      <input
        {...props}}
      />
      <label
        htmlFor={props.id}
      >
        {props.placeholder}
      </label>
      <div>{error}</div>
    </div>
  )
}
```





## React.[Event]Event<[HTMLElement]>

onChange나, onSubmit 의 콜백함수에 사용하면 좋다.

```typescript
// onChange 콜백
const onChangeText = (e: React.ChangeEvent<HTMLInputElement>) => {
    setErrorMsg('')
    setText(e.target.value)
}

// onSubmit 콜백
const onClickComplete = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault()
    if (text.length === 0) {
      setErrorMsg('내용을 입력해주세요')
      return
    }
    const res = onClickCompleteCallback(text)
    if (typeof res === 'string') {
      setErrorMsg(res)
    }
}
```





## as [Type]

어떠한 value의 타입을 한 타입으로 강제로 인식 시키는 방법이다.

```typescript
const handleResize = (e: UIEvent) => {
      const w = e.target as Window
      setScreenSize(calcCurrentScreen(w.innerWidth))
}
```

사실은 매우 지양해야할 방법이다. 하지만, Typing이 너무 오래 걸려 개발 시간에 지장을 끼칠때 어쩔 수 없이 사용한다.







## Axios.isAxiosError

Axios를 사용한 API 통신시 에러가 발생했을 경우, Axios에서 에러를 감싸서 throw하는데, 이때 이것을 다음과 같이 AxiosError라고 인식시켜서 Typing에 문제 없도록 하는 방법이다.

```typescript
if (axios.isAxiosError(err)) {
    switch (err.response?.data.message) {
      case [Strings.ErrorMessageFromServer.EXCEED_NONMEMBER_MAX_PLAYLIST]:
        errorMessage = Strings.ErrorMessageToUser.EXCEED_NONMEMBER_MAX_PLAYLIST
        break
    }
}
```







## children : React.ReactNode

Wrapper 컴포넌트와 같이 다른 컴포넌트를 감싸는 컴포넌트를 만들었을때, children 값을 받아서 렌더해줘야하는데, 이때 children의 속성은 React.ReactNode이다.

```typescript
export default function AppInit({ children }: { children: React.ReactNode }) {
    ...
    return <>{children}</>
}
```


---
title: React 새로고침/뒤로가기 막기
author: 신성일
date: 2023-07-29 22:00:00 +0900
categories: [study, react]
tags: [react]
---

웹페이지에서 사용자가 무언가를 입력하고 뒤로가기를 눌렀을때 다음과 같은 경고창을 띄우는 걸 본적이 있다.

![HTML DOM beforeunload 이벤트 - 제타위키](/assets/img/2023-07-29-react-뒤로가기-막기/Chrome_-_이_사이트에서_나가시겠습니까.png)

이러한 스펙을 개발하기위해서는 다음과 같이 두가지 상황으로 나누어야한다.

- 새로고침/창닫기/링크이동(외부페이지 이동 or 링크 버튼 클릭)
- 뒤로가기

그렇다면 리액트에서는 구체적으로 어떻게 이 스펙을 구현할 수 있을지 알아보자

<br/>

## 새로고침/창닫기/링크이동

새로고침/창닫기/링크이동의 경우는 간단하다. 해당 이벤트들이 발생할때 window 객체에서 발생시키는 `beforeunload` 이벤트를 취소시키면 된다.
리액트에서 구현하려면 다음과 같이 `useEffect` 안에 `window.onbeforeunload` 함수를 등록해주면 된다. 

```react
useEffect(() => {
    window.onbeforeunload = (e) => {
        e.preventDefault();
        return ""
    }

    return () => {
        window.onbeforeunload = null
    }
}, [])
```

하지만, 이 경우에는 페이지에 접속했을 경우 무조건 이벤트가 등록이 된다. 사용자가 입력했을때만 경고창을 띄우고 싶다면 다음과 같이 할 수 있다.

```react
const [hasInputFilled, setHasInputFilled] = useState(false);

useEffect(() => {
    window.onbeforeunload = () => {
        if (hasInputFilled) {
            return ""
        }
    }

    return () => {
        window.onbeforeunload = null
    }
}, [hasInputFilled])
```

<br/>

## 뒤로가기 막기

뒤로가기 이벤트는 `popstate` 이벤트가 발생한다. 하지만 `popstate` 이벤트는 취소가 불가능하다. 따라서 취소를 하지 않고 뒤로가기를 막아야하는데, 방법을 말하기 전에 먼저 `popstate` 이벤트에 대해 간략히 알아보자.

우리가 브라우저에서 여러 웹 페이지를 오가다보면 방문기록이 쌓이게 된다. 이때 하나의 탭에서 여러 웹 페이지를 방문하면 그 기록이 차례로 쌓이고, 뒤로가기 시에는 해당 탭에서 마지막으로 방문했던 곳으로 이동하게 된다. 마치 스택처럼 동작한다. `popstate`는 뒤로가기 시 마지막으로 방문했던 페이지로 이동하기 위해 발생된다.

여기서 뒤로가기를 막기 위한 한가지 아이디어를 찾을 수 있다. 만약 마지막으로 방문했던 페이지가 현재 페이지면, 뒤로가기를 하더라도 현재 페이지에 그대로 남아있을 것이라는 아이디어다. 이를 위해서는 `history.pushState()`를 사용하면 된다.

```react
useEffect(() => {
    const preventGoBack = () => {
        history.pushState(null, "", location.href);
    };
  
    history.pushState(null, "", location.href);
    window.addEventListener("popstate", preventGoBack);

    return () => {
        window.removeEventListener("popstate", preventGoBack);
    }
}, [])
```

1. 마운트 시,`history.pushState()`로 현재 페이지를 스택 마지막에 추가
2. `popstate` 이벤트 콜백 등록
3. `popstate` 이벤트 발생시, 스택이 하나 줄어듦. but 콜백 함수에서 다시 스택을 추가함

하지만 이와같은 방법에는 단점이 있는데, 사용자가 아무리 뒤로가기를 눌러도 뒤로가기가 동작되지 않는 것처럼 보이게 만든다는 점이다. 이를 해소하기 위해 다음과 같이 콜백함수를 수정할 수 있다.

```react
const preventGoBack = () => {
    if(confirm("정말 뒤로 가시겠습니까?")) {
        history.back();
    } else {
        history.pushState(null, "", location.href);
    }
};
```

스택을 하나 추가하기 전에 사용자에게 경고창을 띄우고, 확인을 눌렀을 경우에는 한번 더 뒤로가기를 시도해 이전 페이지로 이동하는 방법이다.

여기서 새로고침/창닫기/링크이동에서 했던 것처럼 사용자가 입력을 했을때만 뒤로가기를 막도록 하면 다음과 같다.

```react
const [hasInputFilled, setHasInputFilled] = useState(false);

useEffect(() => {
    const preventGoBack = () => {
        if(!hasInputFilled) {
          return;
        }
      
        if(confirm("정말 뒤로 가시겠습니까?")) {
            history.back();
        } else {
            history.pushState(null, "", location.href);
        }
    };
  
    if(hasInputFilled) {
        history.pushState(null, "", location.href);
    }
    window.addEventListener("popstate", preventGoBack);

    return () => {
        window.removeEventListener("popstate", preventGoBack);
    }
}, [hasInputFilled])
```

여기서 스택에 넣어진 현재 페이지의 길이를 저장하여 조금 더 안정적으로 동작하도록 할 수 있다.

```react
const [hasInputFilled, setHasInputFilled] = useState(false);
const [pushedLength, setPushedLength] = useState(0)

useEffect(() => {
    const preventGoBack = () => {
        if(pushedLength === 0) {
          return;
        }
      
        if(confirm("정말 뒤로 가시겠습니까?")) {
            history.back();
        } else {
            history.pushState(null, "", location.href);
        }
    };
  
    if(hasInputFilled && pushedLength === 0) {
        history.pushState(null, "", location.href);
        setPushedLength(1)
    } else if(!hasInputFilled && pushedLength === 1) {
        history.back();
        setPushedLength(0);
    }
  
    window.addEventListener("popstate", preventGoBack);

    return () => {
        window.removeEventListener("popstate", preventGoBack);
    }
}, [hasInputFilled, pushedLength])
```



[React](React) 18에서 새로 추가된 Streaming SSR 기술에 대한 정리


## 기존의 SSR 방식

1. 최초 웹 사이트 진입시 서버는 HTML 렌더링에 필요한 데이터들을 불러옴
2. 서버에서 `renderToString`을 통해 렌더링을 진행하고 HTML 파일을 만듬
3. 서버에서 HTML 파일이 다 만들어지면 클라이언트에서 해당 HTML을 받고 자바스크립트까지 모두 로드함
4. 이후 `hydrate` 작업 수행

이러한 방식 때문에 아래와 같은 단점이 있었다

- 서버는 모든 데이터를 다 가져와야 렌더링을 시작할 수 있음
- 클라이언트는 HTML과 모든 자바스크립트를 다운 받아야 hydrate 가능
- hydrate는 모든 컴포넌트를 대상으로 한번에 진행되어야함. 사용자는 이 동안 페이지와 상호작용할 수 없음


## Streaming SSR

가장 빠르게 그릴수 있는 부분은 먼저 렌더링하고 이후는 Node.js의 스트림으로 점진적으로 렌더링하는 방식. `Suspense`, `코드분할`, `스트리밍 HTML`, `선택적 Hydrate`를 이용함

![](/assets/images/Pasted%20image%2020241230201544.png)

- 데이터 fetch가 필요한 부분을 `Suspense`로 감싸서, 데이터 fetch가 필요없는 부분부터 렌더링하고 내려보냄
- 데이터 fetch가 필요한 부분은 데이터가 들어오면 렌더링을 진행하여 내려보냄
- hydrate는 각각 진행됨


### HTML 스트리밍

컴포넌트들을 청크로 분리하고 Suspense로 감싼 뒤에 리액트에게 해당 컴포넌트를 기다리지말고 나머지 페이지먼저 HTML을 내려보내는 기능. Suspense에 감싸진 컴포넌트를 기다리는 동안 fallback UI를 노출할 수 있고, 컴포넌트별로 우선순위를 설정할 수 있음.

이후 해당 컴포넌트의 작업이 완료되면 Stream으로 script와 함께 컴포넌트를 넣어주기에 데이터가 특별한 순서에 맞춰 로딩될 필요도 없다

### 선택적 hydrate

기존에는 전체 HTML을 대상으로 hydrate를 진행했지만 선택적 hydrate를 통해 리액트 코드의 부분부분이 로드될 때마다 해당 부분들을 hydration 한다.


### Streaming SSR이 가능한 이유

HTTP와 Node.js의 Stream API 덕분에 가능하다

HTTP/1.1 에서는 청크 전송 인코딩 방식이 존재한다. 이것은 Response Header에 Transfer-Encoding을 chunked로 전송하면 되는데 일반적으로는 대용량 파일 전송, 실시간 데이터 스트리밍에 사용한다. 이를 활용하여 별도의 socket 같은 방식말고도 지속적으로 데이터를 스트림 형태로 내려보낼 수 있다.

이러한 방식을 보내주는 서버에서는 지원해야하는데, Node.js의 Stream API를 통해 데이터를 작은 청크로 분리하여 ReadableStream 형태로 내려줄 수 있게 되었다. 


---


## 참고
- https://velog.io/@tlsakch510/React-Streaming-SSR-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0
- https://sangminnn.tistory.com/entry/%EB%8D%94-%EC%A2%8B%EC%9D%80-%EC%9C%A0%EC%A0%80-%EA%B2%BD%ED%97%98%EC%9D%84-%EC%9C%84%ED%95%9C-Streaming-SSR
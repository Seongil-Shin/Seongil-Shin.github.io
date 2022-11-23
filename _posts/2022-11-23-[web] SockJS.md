---
title: SockJS
author: 신성일
date: 2022-11-23 21:19:26 +0900
categories: [study, web]
tags: [web, socket, sockjs]
---

# SockJS

주로 Spring을 사용할 때, `WebSocket Emulation` 을 위한 라이브러리이다. 

`WebSocket Emulation` 이란 우선 웹 소켓으로 소켓 연결을 시도하고, **실패할 경우 HTTP Streaming, Long-Polling 같은 HTTP 기반의 다른 기술로 전환해 다시 연결을 시도**하는 것을 말한다. 따라서 다음과 같은 상황에서 도움이 된다

- WebSocket을 지원하지 않는 브라우저
- Server/Client 중간에 위치한 Proxy가 Upgrade 헤더를 해석하지 못해 서버에 전달하지 못한 상황
- Server/Client 중간에 위치한 Proxy가 유휴 상태에서 도중에 connection을 종료시킬 수도 있다.

`WebSocket Emulation`을 위해서 node.js를 사용한다면 Socket.io를 사용하고, Spring을 사용하면 SockJS를 사용하는 것이 일반적이다.

Spring 프레임워크는 Servlet 스택 위에서 Server/Client 용도의 SockJS 프로토콜을 모두 지원한다

<br/>

## SockJS의 구성

- SockJS 프로토콜
- SockJS javascript client : 브라우저에서 사용되는 클라이언트 라이브러리
- SockJS Server 구현 : spring-websocket 모듈을 통해 제공
- SockJS Java Client : spring-weboscket 모듈을 통해 제공 

## WebSocket Emulation Process

SockJS 클라이언트는 서버의 기본 정보를 얻기위해 `GET /info`를 호출하는데, 이는 다음과 같은 정보를 얻기 위함이다

- 서버가 WebSocket을 지원하는지
- 전송과정에서 Cookies 지원이 필요한지
- CORS를 위한 Origin 정보

![info 요청](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbZp2Bu%2Fbtq1OM0cPc2%2F4wVEG1T6fnwKbVwuxQgREK%2Fimg.png)

![info 응답](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcaZXp4%2Fbtq1I8jRmCu%2FNhj5KfKGFVZThMyk0qvWdK%2Fimg.png)

이후 SockJS는 어떤 전송 타입을 사용할 지 결정한 후 아래 와 같은 순서대로 사용하려고 시도한다.

1. WebSocket
   - WebSocket Handshaking을 위한 하나의 HTTP 요청을 더 보낸다.![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdBACTK%2Fbtq1ISnXXlJ%2FIvxcusE04cWtG3cF6SNFq0%2Fimg.png)
   - 그 이후 모든 메세지들은 Socket을 통해 교환된다.
2. HTTP Streaming
   - Ajax/XHR Streaming은 서버 -> 클라이언트로의 메세지들을 위해 하나의 Long-running 요청이 있다.![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb5G3Oq%2Fbtq1KCjXvj6%2FNCoiNv7i3tKL5kt0LFcfOK%2Fimg.png)
   - 추가적인 HTTP POST 요청은 클라이언트 -> 서버로의 메세지를 위해 사용된다.
3. HTTP Long Polling
   - 서버 -> 클라이언트로의 응답 후 현재의 요청을 끝내는 것을 제외하고는 XHR Streaming과 유사하다

<br/>

## 동작

### 전송 요청 형식

모든 전송 요청은 다음의 URL 구조를 갖는다.

> https://host:port/myApp/myEndpoint/{server-id}/{session-id}/{transport}

- server-id : 클러스터에서 요청을 라우팅하는데 사용하나 이외에는 의미 없음
- session-id : SockJS session에 소속하는 HTTP 요청과 연관성 있음
- transport : 전송타입 (ex : websocket, xhr-streaming, xhr-polling)

### SockJS 메세지 최소화

SockJS는 메세지 Frame 크기를 최소화하려고 노력한다. 예를 들어 서버는 다음과 같은 축약 메세지를 보낸다.

- o : open frame. frame을 열 때 사용. 메세지는 `["msg1", "msg2"]`와 같은 JSON-Encoded 배열로서 전달된다., 
- h : heartbeat frame. 25초간 메세지 흐름이 없는 경우에 전송
  - 프록시가 연결이 끊겼다고 결론을 내리는 것을 방지하기 위해 보냄. 기본값은 25초.
  - STOMP를 이용해 heartbeat를 주고 받는 경우 SockJS Heartbeat 설정은 비활성화된다.
- c : close frame. 해당 세션을 종료한다. 

<br/>

## SockJS 사용

### 스프링에서

스프링에서 웹 소켓을 사용하기 위한 의존성인 `spring-websocket`은 SockJS 프로토콜을 모두 지원한다. 따라서 간단히 적용가능한데, 웹소켓을 설정하는 곳에서 다음과 같이 `.withSockJS()`를 사용하면 된다.

```java
@EnableWebSocketMessageBroker
@Configuration
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
	@Override
	public void registerStompEndpoints(StompEndpointRegistry registry) {
		registry.addEndpoint("/ws-stomp")
			.setAllowedOrigins("*")
			.withSockJS();		// SockJS 활성화
	}
}
```

### internet explorer 8 & 9

SockJS는 Ajax/XHR Streaming을 microsoft의 `XDomainRequest(xdr)`를 통해 지원하고 있다. 이는 서로 다른 도메인 간에도 동작하지만, Cookie 전송은 지원하지 않는다. 

Cookie는 Java 기반 서버에서는 필수적이지만, SockJS는 자바만이 아닌 다양한 서버 타입을 위해 고안되었기에 Cookie를 중요하게 다룰지 여부를 알려줘야한다.

SockJS는 처음에 `GET /info` 요청을 날리는데, 이때의 응답으로 `cookie_needed` 속성을 볼 수 있다. 만약 Cookie가 필요하다고 표시되냐 안되냐에 따라 사용하는 기술이 달라진다

- cookie_needed false : XDomainRequest (xdr) 사용
- cookie_needed true : iframe 기반의 기술 사용

스프링에서는 `setSessionCookieNeeded`를 통해 쿠키 사용여부를 표시할 수 있다. 기본값은 true이다.

여기서 iframe 기반의 기술 사용하려면 `X-Frame-Options` 응답 헤더에 대해 알아야한다.

**X-Frame-Options**

- 해당 페이지를 \<frame\> 또는 \<iframe\>, \<object\> 에서 렌더링 할 수 있는지 여부를 나타내는데 사용되고, 사이트내 컨텐츠들이 다른 사이트에 포함되지 않도록 해 clickjacking 공격을 막아내기 위해 사용된다.
- 다음 옵션을 포함
  - deny : 어떤 사이트에서도 frame 상에서 보여질 수 없다
  - sameorigin : 동일한 사이트의 frame에서만 보여진다
  - allow-from uri : 지정된 특정 uri의 frame에서만 보여진다.

 `X-Frame-Options` 속성의 값이 `sameorigin`이나 `allow-from uri`이면 iframe에서 동작된다. `deny`일 경우에는 다음 두가지 SockJS 객체 선언 중 2번째로 선언하였을 때만 동작한다

```javascript
1.
var sockJs = new SockJS("http://localhost:8080/ws/chat");

2.
var sockJs = new SockJS("http://localhost:8080/ws/chat", null, {transports: ["websocket", "xhr-streaming", "xhr-polling"]});
```

### internet explorer 10 이상

IE 10 이상부터는 웹 소켓이 지원된다. 다만 화살표 함수가 지원되지 않아 이부분만 바벨이나 직접 수정으로 해결해주면 된다.

<br/>

## 출처

https://velog.io/@koseungbin/WebSocket
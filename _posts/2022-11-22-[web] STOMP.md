---
title: STOMP
author: 신성일
date: 2022-11-17 18:19:26 +0900
categories: [study, web]
tags: [web]
---

## STOMP(Simple Text Oriented Messaging Protocol)란?

**웹소켓 위에서 동작하는 서브 프로토콜**로, 클라이언트와 서버가 서로 통신하는데 있어 메시지의 형식, 유형, 내용 등을 정의해주는 프로토콜이다.

웹 소켓 프로토콜은 Text 또는 binary 두가지 유형의 메시지 타입을 정의하지만 메시지의 내용에 대해서는 정의하지 않는다. 즉, 웹 소켓만 사용해서 구현하게 되면 해당 메시지가 어떤 요청인지, 어떤 포맷으로 오는지 정해져있지 않아 일일이 구현해야한다.

이때 STOMP라는 프로토콜을 서브 프로토콜로 사용하여, **단순한 binary, text가 아니라 규격을 갖춘 메시지를** 보낼 수 있다.

또한 **기본적으로 pub/sub 구조**로 되어있어 메시지를 전송하고 메시지를 받아 처리하는 부분이 확실히 정해져 있기 때문에 개발자 입장에서 명확하게 인지하고 개발할 수 있는 이점이 있다.

STOMP를 이용하면 메세지의 헤더에 값을 줄 수 있어 헤더 값을 기반으로 통신시 인증 처리를 구현하는 것도 가능하며, STMOP 스펙에 정의한 규칙만 잘 지키면 여러 언어 및 플랫폼간 메시지를 상호 운영할 수 있다.

스프링은 spring-websocket 모듈을 통해서 STOMP를 제공하고 있다.

<br/>

## 형식

### 형식

STOMP는 HTTP에서 모델링되는 Frame 기반 프로토콜이다. Frame은 몇 개의 Test line으로 지정된 구조인데 첫번째 라인은 Text이고, 이후 Key:Value 형태로 Header의 정보를 포함한다. 그 다음 빈 라인을 추가하고 Payload가 존재한다.

아래 형식을 보면 HTTP와 유사하다는 것을 알 수 있다. 

```
COMMAND
header1:value1
header2:value2

Body^@
```

- COMMAND

  - 클라이언트는 메시지를 전송하기 위해 COMMAND로 SEND 또는 SUBSCRIBE 명령을 사용한다
  - MESSAGE COMMAND는 모든 구독자에게 메세지를 브로드캐스팅할 때 사용한다.
  - 그외 다음과 같은 COMMAND가 있다
    - UNSUBSCRIBE, BEGIN, COMMIT, ABORT, ACK, NACK, DISCONNECT
  
- header, value

  - 메시지의 수신 대상과 메시지에 대한 정보를 설명한다.

  - 기존 웹소켓만으로는 표현할 수 없는 형식이다. 

  - destination : 헤더 중 하나로, 이 헤더로 지정한 주소로 메세지를 보내거나 구독할 수 있다.

    - destination은 의도적으로 정보를 불분명하게 정의하였는데, 이는 STOMP 구현체에서 문자열 구문에 따라 직접 의미를 부여하도록 하기 위함이다.

    - 일반적으로 다음의 형식을 따른다.

      ```
      "/topic/..." : publish - subscribe(1:N)
      "/queue/..." : point-to-point (1:1)
      ```


### **메시지 브로커**

publisher로부터 전달 받은 메시지를 subscriber로 전달해주는 중간역할을 수행한다. 동작방식은 다음과 같다.

1. 각각 A, B, C라는 유저가 차례로 5번방에 입장한다.

   ```
   SUBSCRIBE
   destination:/subscribe/chat/room/5
   
   ^@
   ```

   - 유저가 입장하면서 5번방에 대한 구독을 한다.
   - 메시지 브로커는 클라이언트의 SUBSCRIBE 정보를 자체적으로 메모리에 유지한다.

2. A가 5번방에서 채팅을 전송한다.

   ```
   SEND
   content-type:application/json
   destination:/publish/chat
   
   {"chatRoomId":5,"type":"MESSAGE","writer":"clientB"}^@
   ```

3. 5번방 메시지 브로커가 메시지를 받는다.

4. 5번방 메시지 브로커가 5번방 구독자(A, B, C)에게 메시지를 전송한다.

<br/>

## Spring에서 STOMP로 구현

### 웹 소켓에 STOMP, 메시지 브로커 연동

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
      registry.enableSimpleBroker("/subscribe");
      registry.setApplicationDestinationPrefixes("/publish");
    }
    
    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
      registry.addEndpoint("/ws-connection")
              .setAllowedOrigins("*")
              .withSockJS();
    }
}
```

- @EnableWebSocketMessageBroker
  - STOMP를 사용하기 위해 선언하는 애노테이션
- WebSocketMessageBrokerConfigurer
  - 메시지 브로커가 지원하는 웹소켓 메시지 처리를 활성화한다.
- configureMessageBroker()
  - 인 메모리 기반의 Simple Message Broker를 활성화한다. 
    - RabbitMQ, ActiveMQ 같은 외부 메세징 시스템을 사용할 수 있다.
  - 메시지 브로커는 `enableSimpleBroker()`로 설정한 값으로 시작하는 주소의 subscriber들에게 메시지를 전달하는 역할을 한다. 예시에서는 `/subscribe`로 설정하여 `/subscribe`로 들어온 요청은 구독으로 정해진다.
  - 마찬가지로 `setApplicationDestinationPrefixes()`로 설정한 값은 publish를 위해 사용된다, 
- registerStompEndpionts()
  - HandShake와 통신을 담당한 EndPoint를 지정한다.
  - 예시에서는 클라이언트에서 서버로 WebSocket 연결을 하고 싶을 때, `/ws-connection`으로 요청을 보내도록 설정하였다.

### ChatRequest DTO

```java
public class ChatRequest {
    private Long senderId;
    private Long receiverId;
    private Long roomId;
    private String message;
}
```

### Controller

```java
@RestController
@RequiredArgsConstructor
public class ChattingController {
    private final ChattingService chattingService;
    private final SimpMessagingTemplate simpMessagingTemplate;
  
    public ChattingController(ChattingService chattingService, SimpMessagingTemplate simpMessagingTemplate) {
      this.chattingService = chattingService;
      this.simpMessagingTemplate = simpMessagingTemplate;
    }
    
    @MessageMapping("/messages")
    public void chat(@Valid ChatRequest chatRequest) {
      chattingService.save(chatRequest);
      simpMessagingTemplate.convertAndSend("/subscribe/rooms/" + chatRequest.getRoomId(), chatRequest.getMessage());
    }
}
```

- SimpMessagingTemplate

  - `@EnableWebSocketMessageBroker`을 통해서 등록되는 빈이다. 브로커로 메시지를 전달한다.

- @MessageMapping

  - 클라이언트가 SEND 할 수 있는 경로.

  - WebSocketConfig 에서 등록한 `setApplicationDestinationPrefixes()` 주소와 합쳐진다

    `/publish/messages`

### 클라이언트 연동

```javascript
function connect() {
    var socket = new WebSocket('/ws-connection');
    stompClient = Stomp.over(socket);
    stompClient.connect({}, function () {
        setConnected(true);
        stompClient.subscribe('/subscribe/rooms/5', function (greeting) {
            console.log(greeting.body);
        });
    });
}

function sendMessage() {
    stompClient.send("/publish/messages", {}, JSON.stringify({
        'message': $("#name").val(),
        'senderId': 7,
        'receiverId': 14,
        'roomId': 5
    }));
}
```

<br/>

## 출처

https://tecoble.techcourse.co.kr/post/2021-09-05-web-socket-practice/

https://dev-gorany.tistory.com/235

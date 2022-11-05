---
title: Keep-Alive
author: 신성일
date: 2022-11-05 18:51:26 +0900
categories: [study, web]
tags: [web, keepalive]
---

# Keep-Alive

아래와 같이 http 헤더를 보다보면 keep-alive라고 명시된 부분이 보일 때가 있다.

-  `connection : keep-alive`
-  `Keep-Alive:timeout=5, max=1000`

이 `keep-alive`는 무슨 뜻일까 궁금하여 찾아보았다.

`connection : keep-alive`

-  connection은 네트워크 연결을 현재 요청이 종료된 이후에도 유지할 것인가를 설정하는 헤더이다.
-  다음과 같은 값을 지정할 수 있음
   -  close : 한 요청이 종료되면 연결을 끊음. 디폴트
   -  keep-alive : 요청이 종료되어도 연결을 유지한다.

`Keep-Alive:timeout=5, max=1000`

-  송신자가 연결에 대한 타임아웃과 요청 최대 갯수를 어떻게 정했는지 알려주는 헤더
   -  timeout : 연결이 idle한 채로 얼마동안 유지할 것인가
   -  max : 최대 몇개의 요청을 주고 close없이 처리할 것인가

**Keep-alive를 사용하는 이유**

-  http 프로토콜은 tcp을 기반으로 만들어져있다. 그리고 tcp에서 두 호스트가 통신을 할 때는 연결을 맺는 과정이 필요하다. 이 연결을 맺는 과정은 `3-way handshaking`으로 진행되며 총 세번 두 호스트가 통신을 해야 연결이 맺어지며 본래 원하는 통신이 가능해진다. 이렇게 세 번 통신하는 과정은 낭비가 있다.
-  `keep-alive`를 설정해주면 요청이 종료되어도 connection을 끊지 않아 다음 요청 때 한번 맺은 연결을 재사용할 수 있어 서버 요청/응답이 빨라진다.

**단점**

-  만약 서버가 많은 클라이언트로부터 요청을 받아야하고, 모든 클라이언트가 `keep-alive`라면 서버는 모든 요청마다 연결을 유지해야하기에 프로세스 수가 기하급수적으로 늘어나 MaxClient 값을 초과하게 된다.
-  따라서 서버에서는 keep-alive를 지원할 때는 적절히 maxClient나 KeepAliveTimeout을 설정해줘야한다.

**http 1.1에서**

`connection : keep-alive`가 디폴트로 설정되어있다.

**http/2에서**

http/2에서는 keep-alive 기능을 사용할 필요가 없다. 그 이유는 다음과 같다.

http/2에서는 http body가 binary로 전송된다.(과거엔 text로 전송됨). 동시에 이전에는 header-body를 묶은 http message가 전송 최소단위였다면, http/2에서는 **binary frame** 이 전송 최소 단위가 된다. 이렇게 변화한 이유에는 기존 `http 1.x`가 가진 다음과 같은 단점 때문이다.

http 1.x가 가진 단점들

-  본문은 압축이 되지만 헤더는 압축이 되지 않는다.
-  연속된 메세지는 비슷한 헤더구조를 가지는데, 그래도 매번 헤더를 전송해야함
-  다중전송(Multiplexing)이 불가능하다.
   -  multiplexing : 하나의 연결에 여러 요청/응답을 병렬적으로 실어 보내는 것. 병렬적이라는 것은 요청을 보내고 응답을 기다리지 않고 또 다른 요청을 보내는 것을 의미
   -  이는 keep-alive를 사용하더라도 [요청-응답] [요청-응답]으로 이루어지지, [요청-요청-요청-응답-응답-응답]으로 이루어지지 않는다.

이를 해결하기 위해 http/2에서는 하나의 연결에 여러 스트림을 사용하여 multiplexing이 가능하도록 했다. 이것이 가능한 이유는 http/2 패킷을 binary frame 단위로 세분화하여 순서에 상관없이 받는 쪽에서 조립할 수 있도록 설계하였기 때문이다. 이러한 multiplexing 덕분에 http/2 에서는 연결을 유지하기 위한 설정인 `keep-alive`가 더이상 필요없어지게 되었다.

![multiplexing](https://d34smkdb128qfi.cloudfront.net/images/kemptechnologieslibraries/illustrations/loadmasterhttp2multiplexing.png?sfvrsn=4fbe5470_0)

이외에도 http/2에서는 다음과 같은 성능 향상 기법이 적용되었다.

-  헤더 압축
   -  http/1.x에서 매 요청마다 헤더를 보내야했던 것과 다르게, http/2에서는 중복되는 헤더를 압축하여 사용한다.
-  서버 푸쉬
   -  http/1.x에서는 클라이언트가 요청을 하고 서버가 그에 응답을 하는 것이 일반적이었다.
   -  하지만 http/2에서는 서버 푸쉬 기능을 통해 클라이언트가 요청하지 않은 데이터를 서버가 스스로 전송해줄 수 있다.
   -  따라서 만약 `index.html`안에 `style.css`, `script.js`, `image.jpg`가 있다면 http/1.x에서는 매번 [요청-응답]을 반복하며 받아와야했지만, http/2에서는 한번에 내려주어 성능을 개선해준다.
-  우선순위
   -  응답에 대한 우선순위를 정하여 먼저 전송처리 해줄 수 있다.
   -  ex) css는 브라우저 렌더링을 막는데, 이를 최대한 빨리 내보내줘서 브라우저 렌더링을 진행함.

[참고문서](https://goodgid.github.io/HTTP-Keep-Alive/)

[참고문서 2](https://taetaetae.github.io/2017/08/28/apache-keep-alive/)

[참고문서 3](https://jins-dev.tistory.com/entry/HTTP2-%ED%8A%B9%EC%A7%95%EB%93%A4%EC%97%90-%EB%8C%80%ED%95%9C-%EC%A0%95%EB%A6%AC)

[참고문서 4](https://withbundo.blogspot.com/2021/08/httphttp2-http2.html)

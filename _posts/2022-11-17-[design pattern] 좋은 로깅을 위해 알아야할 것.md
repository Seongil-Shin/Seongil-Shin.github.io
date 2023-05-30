---
title: 좋은 로깅을 위해 알아야할 13가지
author: 신성일
date: 2022-11-17 15:23:26 +0900
categories: [study, design pattern]
tags: [design]
---

이 글은 [Logging Best Pratices : The 13 You Should Know](https://www.dataset.com/blog/the-10-commandments-of-logging/)를 기반으로 요약, 학습한 글입니다.

<br/>

# 좋은 로깅을 위해 알아야할 13가지

## Don't write logs by yourself

printf을 사용하거나 파일에 직접 입력하지마라. 로깅을 위한 표준 라이브러리를 사용하라. 

표준 라이브러리를 사용할 때 장점

- 어플리케이션의 다른 부분과 잘 조화됨을 보장할 수 있음
-  올바른 곳에 로그를 남길 수 있음.

표준 라이브러리 종류

- syslog()
- log4j, logback

<br/>

## Log at the proper level

어떤 레벨로 로그를 남겨야할지는 어렵다. 대략적인 팁은 다음과 같다

- TRACE : 프로덕션에서 사용되면 문제가 될 레벨로, 개발 단계에서 버그를 추적할 때 쓰여야한다. VCS에는 커밋하면 안된다
- DEBUG : 프로그램에서 발생하는 어떤 것이든 남기는 레벨이다. 디버깅에 주로 쓰인다.
- INFO : 유저가 일으킨 모든 액션에 대해 남긴다. 또는 스케쥴 동작 같은 시스템 동작에도 남긴다
- NOTICE : error와는 상관없이 프로덕션 상에서 프로그램이 돌아갈 때 알릴만한 이벤트에 대해서 남긴다
- WARN : error로 바뀔 수 있는 모든 이벤트에 남긴다.
- ERROR : 모든 error에 대해 남긴다. 예시로는 API error나 internal error가 있다.
- FATAL : 아주 위험한 이벤트가 발생했을 때 남긴다. 시스템이 네트워크에 연결이 안되거나, 프로그램이 종료되었을 때 남긴다.

<br/>

## Employ the proper log category

대부분의 로깅 라이브러리는 category 지정을 지원한다. 

적절한 카테고리를 지정해서 알아보기 쉽게하자

<br/>

## write meaningful log messages

로그를 읽는 사람이 프로그램에 대해 잘 알고 있음을 가정하고 남기는 로그보다 나쁜 것은 없다. 로그를 남길 때는 항상 긴급 상황을 상상하라.

로그를 작성할 때는 코드 컨텍스트에 맞춰 작성하기 마련이다. 하지만 실제로 로그를 읽어야할 때는 그러한 컨텍스트는 없고, 로그는 이해하기 어려워진다. 

따라서 로그 메세지에 부가정보를 남기든가, 아니면 해당 작업의 목적이 무엇인지, 바람직한 결과는 무엇인지를 남겨라.

또한 이전 로그에 의존한 내용을 작성하지 마라. 이전 로그는 기록되지 않을 수도 있고, 다른 카테고리나 레벨로 기록되었을 수도 있다. 

<br/>

## Write log mesages in english

영어로 작성하면 아스키문자로 작성하였다는 것을 의미한다. 이는 어떤 소프트웨어를 사용하든 잘 작동한다는 것이다. UTF-8으로 작성하였다면 어떤 소프트웨어에서는 잘 작동하지 않을 수도 있다.

만약 영어로 작성하지 않을 때는 의미있는 에러 코드를 앞에 작성하라. 

<br/>

## Add context to your log messages

만약 로그 메세지가 아래와 같다고 해보자.

```
Transaction failed
User operation succeds
java.lang.IndexOutOfBoundsException
```

왜 실패했고 성공했는지 알수가 없다. 이처럼 적절한 컨텍스트가 없는 로그는 노이즈다. 따라서 로그는 아래와 같이 작성되어야한다.

```
Transaction 2346432 failed : cc number checksum incorrect
User 54543 successfully registered e-mail user@domain.com
IndexOutOfBoundsException: index 12 is greater than collection size 10
```

이를 위해선 `MDC`라는 자바 로깅 라이브러리가 사용될 수 있다. MDC는 한 thread에서 공유되는 연관 배열이다. 로거 설정으로 MDC 내용물이 항상 로그에 남도록 설정할 수 있다. 만약 프로그램이 `per-thread paradigm`이라면 context를 남기는 것은 MDC를 이용하면 쉽다.  

다만 MDC는 비동기 로깅 스키마하고는 잘 동작하지 않는다. 

<br/>

## Log in machine parseable format

로그 파일을 사람이 일일히 읽기에 힘들 수도 있다. 따라서 자동화 프로그램으로 로그를 읽으려하는데 파싱하기 어려운 형태면 프로그램을 만들 때 문제가 될 수 있다.

- 파싱하기 어려운 로그

  ```
  2013-01-12 17:49:37,656 [T1] INFO  c.d.g.UserRequest  User 1334563 plays 4 of spades in game 23425656
  ```

- 파싱하기 좋은 로그

  ```
  2013-01-12 17:49:37,656 [T1] INFO  c.d.g.UserRequest  User plays {'user':1334563, 'card':'4 of spade', 'game':23425656}
  ```

<br/>

## But make the logs human-readable as wwell

로그는 파싱하기는 쉬워야하지만 사람이 읽기에도 편해야한다. 사람도 로그를 읽기 때문이다. 

- 표준 date, time 포맷을 사용하라 (ISO8601)
- UTC나 offset을 더한 지역 timestamp를 사용하라
- log level를 적절히 사용해라
- 로그의 세분화를 위해 로그를 서로 다른 타겟에 다른 레벨로 분리하라
- 예외를 로깅할 땐 stack trace를 포함하라
- multi-thread 어플리케이션에선 스레드 이름을 남겨라

<br/>

## Don't log to much or too little

로그 양에는 적절한 밸런스가 중요하다. 이를 달성하기위한 마법의 규칙은 없다. 

대신 처음엔 많이 로그를 남겼다가, 실제 배포 후 분석하여 로그를 줄이거나 늘리는 방법으로 조절할 수 있다. 특히 트러블슈팅을 할 때, 더 늘리거나 줄였으면 하는 부분을 고쳐가며 조절하면 좋다. 이 과정에서 ops와 devs 간의 커뮤니케이션이 필요하다

<br/>

## Think of tour audience

로그를 읽는 사람이 누구냐에 따라 남길 메세지, 컨텍스트, 카테고리, 레벨이 달라진다.

- end-user 일수도 있다 (ex.- 데스크탑 앱일 경우)
- 시스템 관리자일수도 있다.
- 개발자일수도 있다

<br/>

## Don't log for troubleshooting purposes only

로그는 트러블슈팅 이외에 다른 목적으로도 쓰일 수 있다. 

- auditing : 비즈니스 요구사항에 필요할 수 있다. 의미있는 이벤트를 잡아 경영 전략을 세우는데 사용할 수 있다.
- profiling : 로그에 timestamp가 있다면, 프로그램을 profiling하는데 좋은 도구가 될 수 있다. 성능 측정 등의 지표로 사용 될 수 있다.
- statistics : 프로그램의 통계를 만드는데 사용할 수 있다. 

<br/>

## Avoid vendor lock-in

한가지 로깅 라이브러리나 프레임워크에 종속되지 말고, 원할 때 바꿀 수 있어야한다.

로깅 라이브러리를 감싼 wrapper를 만들고, 그 wrapper의 인터페이스로 로그를 남기면 교체를 하고 싶을 때 그 wrapper만 수정하면 되므로 교체가 수월하다.

slf4j 같은 라이브러리는 로깅 프레임워크 교체를 수월하게 해주는 표준화된 추상화를 제공한다.

<br/>

## Don't log sensitive information

password, 카드번호, 주민번호와 같은 민감한 정보는 로그에 남겨선 안된다. 유출되거나 악용되면 큰일나기 때문이다.

무엇을 로그로 남기면 안되는지에 대한 법도 존재한다. 

<br/>

## 출처

https://www.dataset.com/blog/the-10-commandments-of-logging/


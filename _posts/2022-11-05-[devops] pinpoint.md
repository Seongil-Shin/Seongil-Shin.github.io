---
title: pinpoint
author: 신성일
date: 2022-11-05 18:23:26 +0900
categories: [study, devops]
tags: [pinpoint]
---

# Pinpoint

> Pinpoint는 대규모 분산 시스템의 성능을 분석하고 문제를 진단, 처리하는 플랫폼입니다. 2012년 7월에 개발을 시작해 2015년 1월 9일에 오픈소스로 공개했습니다

<br/>

## Pinpoint 개발 동기와 Pinpoint의 특징

인터넷 서비스의 시스템 복잡도가 증가함에 따라 장애나 성능 문제가 발생했을 때 해결이 어려워졌다. 네이버의 시스템도 마찬가지였다. 시스템 복잡도가 높아지며 발생하는 문제를 해결하기 위해 n계층 아키텍처를 효과적으로 추적할 수 있는 새로운 플랫폼을 개발하기로 했고 그것이 Pinpoint이다.

Pinpoint에는 n계층 아키텍처를 추적할 수 있는 다음과 같은 특징이 있다.

-  분산된 애플리케이션의 메시지를 추적할 수 있는 분산 트랜잭션 추적
-  애플리케이션 구성을 파악할 수 있는 애플리케이션 토폴로지 자동 발견
-  대규모 서버군을 지원할 수 있는 수평 확장성
-  코드 수준의 가시성을 제공해 문제 발생 지점과 병목 구간을 쉽게 발견
-  bytecode instrumentation 기법으로 코드를 수정하지 않고 원하는 기능을 추가

<br/>

## Google Dapper 스타일의 분산 트랜잭션 추적

Pinpoint는 Google Dapper 스타일의 추적 방식을 사용해, 분산된 요청을 하나의 트랜잭션으로 추적한다.

### Google Dapper의 분산 트랜잭션 추적방법

분산 트랜잭션 추적의 핵심은 다음 그림처럼 Node 1에서 Node 2로 메시지를 전송했을 때, 분산된 Node 1과 Node 2가 처리한 메시지의 관계를 찾아내는 것이다.

![google dapper 그림](https://d2.naver.com/content/images/2015/06/helloworld-1194202-1.png)

google dapper에서는 Node 1에서 발송하는 메시지에 태그를 추가하여 각 메시지를 구분하도록 하여 추적이 가능하도록 했다.

Pinpoint에서는 Google Dapper 스타일의 추적 방법을 변형해 호출 추적에 사용한다. 원격 호출 시 분산 트랜잭션의 추적을 위해 애플리케이션 수준에서 태그 데이터를 호출 헤더에 추가한다. 태그 데이터는 키의 집합으로 구성되며, 이 집합을 TraceId라고 정의한다.

## Pinpoint 의 자료구조

Span

-  RPC(remote procedure call) 추적을 위한 기본 단위다.
-  RPC가 도착했을 때 처리한 작업을 나타내며 추적에 필요한 데이터가 들어 있다. 코드 수준의 가시성을 확보하기 위해, Span의 자식으로 SpanEvent라는 자료구조를 가지고 있다.
-  Span은 TraceId를 가지고 있다.

Trace

-  Span의 집합으로, 연관된 RPC(Span)의 집합으로 구성된다.
-  Span의 집합은 TransactionId가 같다.
-  Trace는 SpanId와 ParentSpanId를 통해 트리 구조로 정렬된다.

TraceId

-  TransactionId와 SpanId, ParentId로 이루어진 키의 집합이다.
-  TransactionId는 메시지의 아이디이며, SpanId와 ParentId는 RPC의 부모 자식 관계를 나타낸다.
   \- TransactionId(TxId): 분산된 노드를 거쳐 다니는 메시지의 아이디로, 전체 서버군에서 중복되지 않아야 한다.
   \- SpanId: RPC 메시지를 받았을 때 처리되는 작업(job)의 아이디를 정의한다. RPC가 노드에 도착했을 때 생성한다.
   \- ParentSpanId(pSpanId): 호출한 부모의 SpanId를 나타낸다.

## Pinpoint 강의

**Observability 와 APM 필요성**

마이크로 서비스가 많아지면서 서비스의 흐름을 파악하기 어려워졌다. 이러한 상황에서 서비스 전반의 Observability를 확보하기 위해 다음 세가지가 필요해졌다.

-  Trace: 서비스에 request가 들어와서 나갈 때까지의 경로를 추적하는 정보 --> Pinpoint
-  Log : 수많은 서버에 나뉘어진 로그정보. --> NELO
-  Metric : 인프라나 인스턴스의 메트릭 정보 --> NPOT

APM(Application Performance Management)

-  어플리케이션의 가용성과 성능을 관리하고 모니터링하는 시스템
-  pinpoint는 대규모 분산시스템에 적합한 APM 이다.

APM 필요한 이유

-  기존의 문제 해결 방법
   -  문제 발생 -> A 시스템의 문제로 확인 -> 담당자 연결 -> A 시스템의 B 서버 문제로 확인 -> 담당자 연결 -> ... 반복
-  APM
   -  문제 발생 -> APM(pinpoint) 확인 -> 담당자 연결

**Pinpoint 소개**

요청 처리 순서

1. 유저의 요청
2. Pinpoint agent instrumentation 에서 요청을 가로채서 필요한 정보를 주입함
3. 어플리케이션에선 요청에 맞는 작업 수행 후 응답 발송
4. Pinpoint agent instrumentation 에서 응답을 가로채서 정보를 수집 후 비동기로 pinpoint collector로 데이터를 보냄.

bytecode 주입

-  java, node와 같은 언어에서는 bytecode를 주입하여 pinpoint에 필요한 작업을 수행
-  다른 언어에선 SDK를 제공하여 직업 pinpoint를 호출

overload

-  pinpoint로 발생하는 성능저하는 현재 3% 미만
-  지속적으로 관찰하여 pinpoint의 성능을 모니터링한다.

**pinpoint feature 소개**

server map

-  pinpoint로 모니터링하는 서버의 구조를 보여줌

scatter chart

-  초록색 : 성공한 transaction, 빨간색 : 실패한 transaction
-  세로축은 응답 시간.
-  드래그하면 선택된 점들의 정보가 보여짐

call stack

-  transaction 하나를 선택하면 상태 콜스택이 보여짐
-  해당 transaction이 어떻게 호출되었는지 확인할 수 있다.

inspector

-  인스턴스의 기본 정보 및 메트릭 정보를 확인가능

[참고문서](https://d2.naver.com/helloworld/1194202)

[Pinpoint 강의](https://share.navercorp.com/cm-4/lecture/327277?isDesc=false)

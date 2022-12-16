---
title: istio
author: 신성일
date: 2022-11-05 18:21:26 +0900
categories: [study, devops]
tags: [istio, devops, msa]
---

# Istio

> istio는 service mesh를 위한 tool입니다. istio를 사용하면 MSA(Micro Service Architecture)를 적용할 때 반복적으로 설정해야하는 L7/모니터링을 쉽고 간편하게 설정할 수 있습니다.

## Service mesh

Service mesh는 API 등을 사용하여 마이크로 서비스 간 통신을 안전하고, 빠르고, 신뢰할 수 있게 만들기 위해 설계된 전용 인프라 계층이다.

> Service mesh는 보통 application 서비스에 경량화 프록시를 사이드카 방식으로 배치하여 서비스 간 통신을 제어하며, Service Discovery, Load Balancing, Dynamic Request Routing, Circuit Breacking, Retry and Timeout, TLS, Distributed Tracing, Metric 수집, Access Control, A/B Testing 기능 등을 지원한다.

MSA 시스템에서 수십개의 마이크로 서비스가 동작할 수 있기에 관리하기가 매우 복잡하다. 또한 인스턴스가 수행되는 네트워크 간의 레이턴시, 신뢰성, 안전성 등을 보장할 수도 없다. 이러한 문제점들은 어플리케이션 계층(7계층)에서 해결할 수도 있지만, 그렇게 되면 application 언어 및 런타임에 종속성이 생기고, 기능 변경 등을 할 때마다 비용이 발생한다.

따라서 MSA 시스템에서는 인프라 레벨에서 별도의 소스 수정 없이 안정적으로 관리할 수 있는 Service mesh가 필요하다.

<br/>

## istio 정의

istio는 Service mesh를 구현할 수 있는 오픈소스 솔루션이다. istio를 사용하면 서비스 코드 변경 없이 로드밸런싱, 서비스 간 인증, 모니터링 등을 적용하여 마이크로 서비스를 쉽게 관리할 수 있다.

마이크로 서비스 간의 모든 네트워크 통신을 담당할 수 있는 프록시인 Envoy를 사이드카 패턴으로 마이크로 서비스들에 배포한다음, 프록시들의 설정값 저장 및 관리/감독을 수행하고, 프록시들에 설정값을 전달하는 컨트롤러 역할을 수행한다.

각 마이크로 서비스에 사이드카 패턴으로 배포된 envoy 프록시를 `Data plane`이라고 하고, data plane을 컨트롤하는 부분을 `control plane`이라고 한다.

<br/>

## istio 구조/구성요소

![arch](https://istio.io/latest/docs/ops/deployment/architecture/arch.svg)

**Data plane**

envoy proxy 세트로 구성되어있다. envoy는 사이드카 방식으로 각각의 마이크로 서비스에 배포되어 서비스로 들어오고 나가는 모든 트래픽을 통제하게 된다. envoy를 통해서 서비스를 호출할 때 호출하는 서비스의 IP주소는 파일럿(Pilot)에 저장된 엔드포인트 정보를 활용한다.

**Control Plane**

Data plane의 Envoy를 컨트롤하는 부분이다. Control plane은 Istiod라는 하나의 모듈로 되어있다.(istio 1.4 버전까지는 별도로 되어있었음)

-  Pilot : Envoy 설정 관리를 수행하는 모듈. Envoy가 호출하는 서비스의 주소를 얻을 수 있는 서비스 디스커버리 기능을 제공. 서비스 트래픽 라우팅 기능 제공. 서비스간 호출 시 재시도(retry), 서킷브레이커, 타임아웃 등의 기능 제공
-  Citadel : 보안 관련 기능을 수행. 사용자 인증, 인가 등을 통한 서비스/엔드 유저 간 인증 강화. TLS(SSL)을 이용한 통신 암호화 및 인증서 관리
-  Galley : istio의 구성 및 설정 검증, 배포 관리 수행.

<br/>

## Istio 주요 특징

**트래픽 관리**

istio의 간편한 규칙 설정과 트래픽 라우팅 기능을 통해 서비스 간의 트래픽 흐름과 API 호출을 제어할 수 있다. 또한 서킷 브레이커, 타임아웃, 재시도 기능과 같은 서비스 레벨의 속성 구성을 단순화하고, 백분율 기반으로 트래픽을 분할하여 A/B test, canary rollout, staged rollout 과 같은 작업을 쉽게 설정할 수 있다.

**보안**

istio는 기본적인 보안 통신 채널을 제공하며, 대규모 서비스 통신의 인증, 권한부여, 암호화 등을 관리한다. istio를 사용하면 기본적으로 서비스 통신은 보호되기 때문에, 다양한 프로토콜이나 런타임에서 어플리케이션 변경을 거의하지 않고 일관된 정책을 시행할 수 있다. 쿠버네티스 네트워크 정책과 함께 사용하여 pod 간 통신을 보호하는 기능 등의 이점이 있다.

**관찰 가능성**

트레이싱, 모니터링, 로깅 기능으로 서비스 메쉬에 배포된 서비스들에 대해 더욱 자세히 파악할 수 있다.

**플랫폼 지원**

istio는 플랫폼에 독립적이며 다양한 환경에서 실행되도록 설계되었다.

**통합과 사용자 정의**

istio의 구성 요소들은 확장 및 커스터마이징을 통해 ACL, 로깅, 모니터링, 할당량, 감사 등을 위한 기존 솔루션과 통합될 수 있다.

## 요약
istio는 마이크로 서비스에서 L7 모니터링, 서비스간 인증, 로드밸런싱을 쉽고 빠르게 붙일 수 있는 오픈소스 솔루션이다.
마이크로 서비스 간 통신을 위해 설계된 인프라계층인 service mesh를 구현한다.
각 마이크로 서비스에는 사이드카 패턴으로 배포된 envoy 프록시가 있고, 이 envoy 프록시들을 Data plane이라고 한다. 그리고 Data plane을 컨트롤 하는 부분을 control plane이라고 한다.
envoy 프록시는 사이드카 방식으로 각 마이크로 서비스에 들어오고 나가는 모든 트래픽을 통제한다. 

<br/>

[참고문서 1](https://twofootdog.tistory.com/78)

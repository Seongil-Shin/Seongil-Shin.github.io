---
title: Canary 테스트
author: 신성일
date: 2022-11-10 18:20:26 +0900
categories: [study, devops]
tags: [devops, canary]
---

## [devops] Canary 테스트

안정적인 버전을 릴리즈하기 전에 테스트 버전을 일부 사용자에게 배포하는 것을 말한다.

만약 카나리 버전에 심각한 버그가 발생한다해도 사용하는 사용자가 적기 때문에 피해를 최소화할 수 있다. 또한 안정적인 버전과 테스트 버전이 모두 배포된 상태이기 때문에 A/B 테스트가 가능하다.

![canary_testing](https://cdn.ttgtmedia.com/rms/onlineImages/canary_testing.jpg)

유저가 직접 카나리 버전을 설치하는 경우도 있지만, 유저는 모르고 사용하도록 할 수 있다. 일부의 유저만 새로운 버전으로 서비스를 제공하는 방식이다.

<br/>

## 출처

https://codechacha.com/ko/what-is-canary-development-test/
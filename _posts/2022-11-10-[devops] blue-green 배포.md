```
title: blue-green 배포
author: 신성일
date: 2022-11-10 18:19:26 +0900
categories: [study, devops]
tags: [devops, blue, green]
```

# **[devops] blue-green 배포**

애플리케이션의 이전 버전에 있던 사용자 트래픽을 이전 버전(blue)과 거의 동일한 새 버전(green)으로 점진적으로 이전하는 애플리케이션 배포 모델이다. 이때 두 버전 모두 프로덕션 환경에서 실행 상태를 유지한다.

blue에서 green으로 완전히 이전되면 blue는 롤백에 대비하여 대기 상태에 두거나 프로덕션에서 가져온 후 업데이트하여 다음 업데이트의 템플릿으로 삼을 수 있다.

k8s는 클라우드 네이티브 애플리케이션의 마이크로서비스를 패키지화하는 컨테이너의 오케스트레이션을 지원하며, 개발자가 애플리케이션 아키텍처를 처음부터 새로 구축할 필요 없이 재사용할 수 있는 아키텍쳐 패턴 컬렉션은 이러한 k8s를 지원한다.

이러한 k8s 패턴 중 **선언적 배포 패턴**이 가장 유명하다. 선언적 배포 패턴을 사용하면 k8s 아키텍처에서 pod를 배포할 때 수작업의 부담이 줄어든다. 



<br/>

## **출처**

https://www.redhat.com/ko/topics/devops/what-is-blue-green-deployment
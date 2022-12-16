---
title: kubernetes
author: 신성일
date: 2022-11-30 18:19:26 +0900
categories: [study, devops]
tags: [k8s, kubernetes]
---

이 포스트는 [subicura 님의 kubenetes 게시글](https://subicura.com/2019/05/19/kubernetes-basic-1.html)을 읽으며 요약/기록한 게시글입니다.

# Kubenetes

쿠버네티스(kubenetes, k8s)는 도커 컨테이너를 쉽고 빠르게 배포/확장하고 관리를 자동화해주는 오픈소스 플랫폼이다.
단순한 컨테이너 플랫폼이 아닌 마이크로 서비스, 클라우드 플랫폼을 지향하고 컨테이너로 이루어진 것들을 손쉽게 담고 관리할 수 있는 그릇 역할을 한다.

<br/>

## 쿠버네티스 특징

### 다양한 배포방식

쿠버네티스는 다음과 같은 다양한 배포방식을 지원한다.

- Deployment : 새로운 버전의 매플리케이션을 다양한 전략으로 무중단 배포할 수 있다
- StatefulSets : 실행 순서를 보장하고 호스트 이름과 볼륨을 일정하게 사용할 수 있어 순서나 데이터가 중요한 경우에 사용
- DaemonSet : 로그나 모니터링 등 모든 노드에 설치가 필요한 경우에 사용
- Job : 배치성 작업에 사용
- CronJob : 배치성 작업에 사용

## ingress 설정

다양한 웹 어플리케이션을 하나의 로드 밸런서로 서비스하기 위해 ingress 기능을 제공한다.

웹 어플리케이션은 보통 외부에서 직접 접근할 수 없도록 애플리케이션을 내부망에 설치하고, 외부에서 접근이 가능한 ALB나 nginx, Apache를 프록시 서버로 활용한다. 프록시 서버는 도메인과 Path 조건에 따라 등록된 서버로 요청을 전달하는데 서버가 바뀌거나 IP가 변경되면 매번 설정을 수정해야한다.

쿠버네티스의 ingress는 이를 자동화하여 기존 프록시 서버에서 사용하는 설정을 거의 그대로 사용할 수 있다. 새로운 도메인을 추가하거나 업로드 용량을 제한하기 위해 일일히 프록시 서버에 접속하여 설정할 필요가 없다.

![ingress](https://subicura.com/generated/assets/article_images/2019-05-19-kubernetes-basic-1/ingress-1000-6431193ae.webp)

하나의 클러스터에 여러 개의 Ingress 설정을 할 수 있어 관리자 접속용 Ingress와 일반 접속용 Ingress를 따로 관리할 수 있다.

### 클라우드 지원

쿠버네티스는 Cloud Controller를 사용하여 클라우드 연동을 손쉽게 확장할 수 있다. AWS 등 많은 클라우드 업체에서 모듈을 제공하여 관리자는 동일한 설정파일을 서로 다른 클라우드에서 동일하게 사용할 수 있다.

### Namespace & label

하나의 클라스터를 `namespace`에 따라 논리적으로 구분하여 사용할 수 있다. 더 세부적인 설정으로 `Label` 기능을 사용하여 유연하고 확장성 있게 리소스를 관리할 수 있다.

### RBAC(Role-Based Access Control)

각각의 리소스에 대해 유저별로 세부적인 권한을 손쉽게 지정할 수 있다. AWS의 경우 IAM을 연동할 수 있다.

### CRD (Custom Resource Definition)

필요한 기능이 쿠버네티스에서 제공하지 않을 수 있다. 이럴때 그러한 기능을 쿠버네티스 기본 기능과 동일한 방식으로 적용하고 사용할 수 있다. 

예를 들어 쿠버네티스는 기본적으로 SSL 인증서 관리기능을 제공하지 않지만 [cert-manager](https://github.com/cert-manager/cert-manager)를 설치하고 Certificate 리소스를 사용하면 쿠버네티스 명령어로 인증서를 관리할 수 있다. 

### Auto scaling

CPU, Memory 사용량, 현재 접속자 수와 같은 메트릭으로 Auto Scaling을 할 수 있다.

컨테이너 개수를 조정하는 Horizontal Pod Autoscaler, 컨테이너의 리소스 할당량을 조정하는 Vertical Pod Autoscaler, 서버 개수를 조정하는 Cluster Autoscaler 방식이 있다. 

### Federation, Multi Cluster

클라우드에 설치한 쿠버네티스 클러스터와 자체 서버에 설치한 쿠버네티스를 묶어서 하나로 사용할 수 있다. 구글에서 발표한 [Anthos](https://cloud.google.com/anthos?hl=ko)를 이용하면 한 곳에서 여러 클라우드의 여러 클러스터를 관리할 수 있다.

<br/>

## 쿠버네티스 기본 개념

### Desired State

`Desired State`는 관리자가 원하는 시스템의 환경을 의미한다.(ex : 얼마나 많은 웹서버가 동작하면 좋은지, 몇번 포트로 서비스되면 좋은지 등)

![desired state](https://subicura.com/generated/assets/article_images/2019-05-19-kubernetes-basic-1/desired-state-1000-9c708dab6.webp)

쿠버네티스는 현재 상태를 모니터링하면서 관리자가 설정한 원하는 상태를 유지하려고 내부적으로 작업하는 로직을 가지고 있다. 이러한 개념 때문에 쿠버네티스는 관리자가 서버를 배포할 때, 직접적인 동작을 명령하지 않고, 상태를 선언하는 방식을 사용한다.

### kubernetes object

쿠버네티스는 상태를 관리하기 위한 대상을 오브젝트로 정의한다. 기본으로 수십가지 오브젝트를 제공하고 새로운 오브젝트를 추가하기 쉽기 떄문에 확장성이 좋다. 주요 오브젝트는 다음과 같다.

- Pod

  <img src="https://subicura.com/generated/assets/article_images/2019-05-19-kubernetes-basic-1/pod-958-c70f76ba0.webp" alt="pod" style="zoom:50%;" />

  - 쿠버네티스에서 배포가능한 가장 작은 단위로, 한 개 이상의 컨테이너와 스토리지, 네트워크 속성을 가진다.
  - Pod에 속한 컨테이너는 스토리지와 네트워크를 공유하고 서로 localhost로 접근할 수 있다. 컨테이너를 하나만 사용하는 경우도 반드시 Pod으로 감싸서 관리한다. 

- ReplicaSet

  <img src="https://subicura.com/generated/assets/article_images/2019-05-19-kubernetes-basic-1/replicaset-692-4745158af.webp" alt="replicaSet" style="zoom:50%;" />

  - 같은 Pod을 여러 개 복제하여 관리하는 오브젝트이다. 
  - Pod을 생성하고 개수를 유지하려면 반드시 ReplicaSet을 사용해야한다.
  - RespicaSet은 복제할 개수, 개수를 체크할 라벨 선택자, 생성할 Pod의 설정값 등을 가지고 있다. 직접적으로 ReplicaSet을 사용하기보단 Deployment 등 다른 오브젝트에 의해서 사용되는 경우가 많다.

- Service

  <img src="https://subicura.com/k8s/build/imgs/guide/service/lb.webp" alt="service" style="zoom:50%;" />

  - 네트워크와 관련된 오브젝트로, Pod을 외부 네트워크와 연결해주고 여러 개의 Pod을 바라보는 내부 로드밸런서를 생성할 때 사용한다. 
  - 내부 DNS에 서비스 이름을 도메인으로 등록하기 때문에 서비스 디스커버리 역할도 한다.

- Volume

  ![volume](https://phoenixnap.com/kb/wp-content/uploads/2021/04/kubernetes-persistent-volumes-1.png)

  - 저장소와 관련된 오브젝트로, 호스트 디렉터리를 그대로 사용할 수 있고 [EBS](https://aws.amazon.com/ko/ebs/) 같은 스토리지를 동적으로 생성하여 사용할 수 있다.
  - 다양한 [저장방식](https://kubernetes.io/docs/concepts/storage/#types-of-volumes)을 지원한다.

### 오브젝트 명세 -YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

오브젝트 명세(Spec)은 YAML 파일로 정의하고, 여기에 오브젝트 종류와 원하는 상태를 입력한다.
이러한 명세는 생성, 조회, 삭제로 관리할 수 있기 때문에 REST API로 쉽게 노출할 수 있다. 
접근 권한 설정도 같은 개념을 적용하여 누가 어떤 오브젝트에 어떤 요청을 할 수 있는지도 정의할 수 있다.

### 쿠버네티스 배포방식

쿠버네티스는 애플리케이션을 배포하기 위해 원하는 상태(desired state)를 다양한 오브젝트에 라벨을 붙여 정의(yaml)하고 API 서버에 전달하는 방식을 사용한다.

“컨테이너를 2개 배포하고 80 포트로 오픈해줘”라는 간단한 작업을 위해 다음과 같은 구체적인 명령을 전달해야한다.

> “컨테이너를 Pod으로 감싸고 type=app, app=web이라는 라벨을 달아줘. type=app, app=web이라는 라벨이 달린 Pod이 2개 있는지 체크하고 없으면 Deployment Spec에 정의된 템플릿을 참고해서 Pod을 생성해줘. 그리고 해당 라벨을 가진 Pod을 바라보는 가상의 서비스 IP를 만들고 외부의 80 포트를 방금 만든 서비스 IP랑 연결해줘.”

<br/>

## 쿠버네티스 아키텍처

쿠버네티스는 중앙(Master)에 API 서버와 상태 저장소를 두고 각 서버(Node)의 에이전트(kubelet)와 통신하는 단순한 구조이다. 하지만 앞에서 얘기한 개념을 여러 모듈로 쪼개어 구현하고, 다양한 오픈소스를 사용하기 떄문에 설치가 까다롭고 구성이 복잡해보이는 것이다.

### 마스터 - 노드 구조

![master-node](https://subicura.com/generated/assets/article_images/2019-05-19-kubernetes-basic-1/master-node-1000-6733a2c1e.webp)

쿠버네티스는 전체 클러스터를 관리하는 마스터와 컨테이너가 배포되는 노드로 구성되어있다. 모든 명령은 마스터의 API 서버를 호출하고 노드는 마스터와 통신하면서 필요한 작업을 수행한다. 특정 노드의 컨테이너에 명령하거나 로그를 조회할 때도 노드에 직접 명령하는게 아니라, 마스터에 명령을 내리고 마스터가 노드에 접속하여 대신 결과를 응답한다.

### 마스터

마스터 서버는 다양한 모듈이 확장성을 고려하여 기능별로 쪼개져있다. 관리자만 접속할 수 있도록 보안 설정을 해야하고, 마스터 서버가 죽으면 클러스터를 관리할 수 없기 때문에 보통 3대를 구성하여 안정성을 높인다.

AWS EKS 같은 경우 마스터를 AWS에서 자체 관리하여 안정성을 높였고(마스터에 접속 불가), 개발환경이나 소규모 환경에서는 마스터와 노드를 분리하지 않고 같은 서버에 구성하기도 한다.

마스터의 구성요소는 다음과 같다.

<img src="https://subicura.com/generated/assets/article_images/2019-05-19-kubernetes-basic-1/kubernetes-master-1000-0b8b00bcb.webp" alt="master components" style="zoom: 67%;" />

- API 서버 kube-apiserver
  - 쿠버네티스의 모든 요청을 처리하는 마스터의 핵심 모듈
  - 실제로 하는 일을 요약하면 **원하는 상태를 key-value 저장소(etcd)에 저장하고, 저장된 상태를 조회**하는 작업
  - kubectl의 요청, 내부 모듈의 요청 처리, 권한 체크, 노드에서 실행중인 컨테이너의 로그를 보여주고 명령을 보내는 디버거 역할 등 수행
    - kubectl : 마스터의 API 서버는 json 또는 protobuf 형식을 이용한 http 통신을 지원한다. 하지만 직접 http 통신으로 명령을 내리면 불편하므로, 보통 `kubectl` 이라는 명령행 도구를 사용한다
- 분산 데이터 저장소 etcd
  - key-value 저장소로, 클러스터의 모든 설정/상태 데이터가 저장되는 곳. 
  - RAFT 알고리즘을 사용하였고, 여러 개로 분산하여 복제할 수 있다. watch 기능을 포함하여 상태 변경 시 바로 체크하여 로직을 실행할 수 있다.
  - 나머지 모듈은 stateless하게 동작하기에 etcd만 안전하게 백업해두면 언제든지 클러스터를 복구할 수 있다. 
  - 오직 API 서버와 통신하고, 다른 모듈은 API 서버를 거쳐 etcd 데이터에 접근한다.
- 스케줄러, 컨트롤러
  - 현재 상태를 모니터링하다가 원하는 상태와 다르면 각자 맡은 작업을 수행하고 상태를 갱신한다.
  - API 서버는 요청을 받으면 etcd 저장소와 통신하고 실제로 상태를 바꾸는 건 스케줄러와 컨트롤러이다.
  - 구성요소
    - 스케줄러 : 할당되지 않은 Pod을 여러가지 조건(필요한 자원, 라벨)에 따라 적절한 노드 서버에 할당시켜주는 모듈
    - 큐브 컨트롤러(kube-controller) : 쿠버네티스에 있는 거의 모든 오브젝트의 상태를 관리. 
      - 오브젝트별로 철저하게 분업화되어 Deployment는 ReplicaSet을 생성하고 ReplicaSet은 Pod을 생성하고 Pod은 스케줄러가 관리하는 식
    - 클라우드 컨트롤러 : AWS 등 클라우드에 특화된 모듈. 노드를 추가/삭제하고 로드밸런서를 연결하거나 볼륨을 붙일 수 있다. 각 클라우드 업체에서 인터페이스를 구현하여 확장성이 좋고, 자체 모듈을 만들어 제공한다.

### 노드

노드 서버는 마스터 서버와 통신하면서 필요한 Pod을 생성하고 네트워크와 볼륨을 설정한다. 실제 컨테이너들이 생성되는 곳이며 수백, 수천대로 확장할 수 있다. 각각의 서버에 라벨을 붙여 사용목적을 정의할 수 있다.

노드의 구성요소는 다음과 같다

<img src="https://subicura.com/generated/assets/article_images/2019-05-19-kubernetes-basic-1/kubernetes-node-1000-f56fb7943.webp" alt="node-components" style="zoom:67%;" />

- Kubelet
  - 노드에 할당된 pod의 생명주기를 관리한다.
  - pod을 생성하고 pod 안의 컨테이너에 이상이 없는지 확인하면서 주기적으로 마스터에 상태를 전달한다.
  - API 서버의 요청을 받아 컨테이너의 로그를 전달하거나 특정 명령을 대신 수행해준다.
- kube-proxy
  - pod으로 연결되는 네트워크를 관리한다
  - TCP, UDP, SCTP 스트림을 포워딩하고 여러 개의 Pod을 라운드로빈 형태로 묶어 서비스를 제공할 수 있다
  - iptables를 설정하여 각 pod으로 요청을 보내준다. 하지만, iptables에 등록된 규칙이 많아지면 느려지는 문제 때문에 최근엔 IPVS를 지원하기 시작했다.

### Node vs cluster

쿠버네티스를 학습하다보면 노드와 클러스터란 단어를 많이 접할 수 있다. 이 둘의 차이는 무엇일까?

- 노드 : 하나의 머신이나 서버. Pod들을 포함함.
- 클러스터 : 서로 높은 가용성으로 통신하는 노드들의 집합.

### 추상화

쿠버네티스는 도커 컨테이너 이외에도 CRI(Container Runtime Interface)를 구현한 다양한 컨테이너 런타임을 지원한다. 또한 CNI(네트워크), CSI(스토리지)를 지원하여 인터페이스만 구현한다면 쉽게 확장하여 사용할 수 있게 제공한다.

<br/>

## 하나의 Pod이 생성되는 과정

관리자가 애플리케이션을 배포하기 위해 ReplicaSet을 생성하면 다음과 같은 과정을 거쳐 Pod을 생성한다.

![order](https://subicura.com/generated/assets/article_images/2019-05-19-kubernetes-basic-1/create-replicaset-1000-9947c6c3a.webp)

각 모듈은 서로 통신하지 않고 오직 API Server하고만 통신한다. API Server를 통해 etcd에 저장된 상태를 체크하고, 현재 상태와 원하는 상태가 다르면 필요한 작업을 수행한다. 각 모듈을 살펴보자

**kubectl**

- ReplicaSet 명세를 yml 파일로 정의하고 kubectl 도구를 이용하여 API Server에 명령을 전달
- API Server는 새로운 ReplicaSet Object를 etcd에 저장

**kube controller**

- Kube Controller에 포함된 ReplicaSet Controller가 ReplicaSet을 감시하다가 ReplicaSet에 정의된 Label Selector 조건을 만족하는 Pod이 존재하는지 체크
-  해당하는 Label의 Pod이 없으면 ReplicaSet의 Pod 템플릿을 보고 새로운 Pod(no assign)을 생성. 생성은 역시 API Server에 전달하고 API Server는 etcd에 저장

**Scheduler**

-  Scheduler는 할당되지 않은 Pod이 있는지 체크
-  할당되지 않은 Pod이 있으면 조건에 맞는 Node를 찾아 해당 Pod을 할당

**Kubelet**

-  Kubelet은 자신의 Node에 할당되었지만 아직 생성되지 않은 Pod이 있는지 체크
-  생성되지 않은 Pod이 있으면 명세를 보고 Pod을 생성
-  Pod의 상태를 주기적으로 API Server에 전달

<br/>

## 출처

https://subicura.com/2019/05/19/kubernetes-basic-1.html

https://subicura.com/k8s/guide/#%E1%84%8B%E1%85%AF%E1%84%83%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%A6%E1%84%89%E1%85%B3-%E1%84%87%E1%85%A2%E1%84%91%E1%85%A9

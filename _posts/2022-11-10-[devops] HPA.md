---
title: HPA
author: 신성일
date: 2022-11-10 18:23:26 +0900
categories: [study, devops]
tags: [kubernetes]
---

# **[devops] HPA**

HorizontalPodAutoscaler의 약자로, k8s에서 CPU 사용률을 체크하여 Pod의 개수를 스케일링하는 기술이다. 지정한 메트릭을 controller가 체크하여 부하에 따라 필요한 pod의 replica 수가 되도록 자동으로 pod 수를 늘리거나 줄일 수 있는 기술이다.

<br/>

## **Auto scaling**

Auto scaling은 사용자가 정의한 주기(스케줄링)나 이벤트(모니터링 알람)에 따라 서버를 자동으로 생성하거나 삭제한다. 서비스 트래픽에 따라 서버를 늘리거나 줄여 발생하는 요금을 최소화한다.

![0*FW7GEykC2wItvKxP](https://miro.medium.com/max/1400/0*fMKxP-l3N6lO97rV)

k8s에서는 다음과 같은 알고리즘을 통해 적절한 pod 개수를 유지한다.

- Horizontal Pod Autoscaler 컨트롤러는 원하는(desired) 메트릭 값과 현재 메트릭 값 사이의 비율로 작동한다.
- 원하는 replica 수 = ceil[현재 replica 수 * (현재 메트릭 값 / 원하는 메트릭 값)]
  - ex) 현재 메트릭 값이 200m이고, 원하는 값이 100m인 경우 replica 수는 두배가 된다. 만약 현재 값이 50mdㅣ면 replica 수는 절반이 된다.

<br/>

## **Scale 정책 설정**

**Deployment 설정 예시**

초기 Replica(Pod) 개수를 1개로 설정

![1*rRw2_cr-91eWD15rk-NViQ](https://miro.medium.com/max/1222/1*rRw2_cr-91eWD15rk-NViQ.webp)

**HPA 설정 예시**

리소스 상태에 따라서 10개까지 scaling 가능하도록 설정

![1*gpX_9eCEwnIXlGxWxOe9tw](https://miro.medium.com/max/1340/1*gpX_9eCEwnIXlGxWxOe9tw.webp)

<br/>

## **메트릭 수집 방법**

auto scaling을 위해서는 k8s에서 서버 메트릭 정보를 수집할 수 있어야한다. k8s에서는 [metric server](https://github.com/kubernetes-sigs/metrics-server)를 통해 pod, node에 대한 리소스 자원들에 대한 metric 정보들을 수집한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FbZcysz%2FbtroRZrd7dd%2FHxk56OA8ZF7x4UqUPpP1q1%2Fimg.png)

Metric server가 수집한 metric 정보를 k8s aggregator에게 노티하는 방식으로 metric 정보가 관리된다. 만약 metric 정보를 얻고 싶다면 별도의 tool을 이용해서 aggregator에 있는 metric 정보들을 가져가게 된다.

이러한 metric server는 기본적은 k8s component에 포함되어있지 않기에 별도의 적용이 필요하다.

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

정상적으로 설치됐다면 `kubectl top` 명령어를 사용할 수 있다. 이 명령도 마찬가지로 metric 정보를 수집한 것을 바탕으로 보여준다. 

<br/>

## **VPA(Vertical Pod Autoscaler)**

k8s에서 metric 정보를 기반으로 auto scaling을 지원해주는 component에는 HPA 외에도 VPA가 있다. 둘의 차이점은 다음과 같다.

- HPA : Scale out에 대한 기능 제공. pod가 배포되는 개수를 높여준다.
- VPA : Scale up에 대한 기능 제공. pod의 리소스 할당량을 높여준다.

VPA에서는 metric을 기준으로 리소스를 늘려야한다고 판단되면, 기존 pod의 리소스 할당량을 높여주는 방식으로 작동한다. 

```java
recommendation:
	containerRecommendations:
	- containerName: nginx
  	lowerBound:
    	cpu: 40m
    	memory: 3100k
  	target:
    	cpu: 60m
    	memory: 3500k
  	upperBound:
    	cpu: 831m
    	memory: 8000k
```

하지만 여기에는 큰 단점이 있다. 바로 리소스 제한을 변경시키기 위한 방법이 **Restart만 존재** 한다는 점이다. 즉, 서비스 중인 pod가 운영정지될 수도 있다는 말이다. 따라서 HPA와 같이 기존 pod는 유지하되 새로운 pod를 추가하여 서비스를 중단시키지 않는 방식이 선호된다.

<br/>

## k8s traffic 분산 방법

그럼 k8s에서는 어떻게 클라이언트 트래픽을 여러 pod들로 분산할까?

[kubernetes 공식문서](https://kubernetes.io/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies)에 따르면, k8s는 iptable를 사용하여 pod들로 트래픽을 분산한다. 이때 발생하는 일은 다음과 같다.

1. 사용자가 `kind:service` object를 생성한다
2. k8s는 virtual clusterIP를 만들고, `kube-proxy deamonset`에게 iptable을 업데이트하라고 지시한다.
3. virtual clusterIP로 온 요청은 pod ip로 로드밸런싱 된다.

로드밸런싱 policy는 기본적으로 round robin 방식이지만, 100% 정확한 표현은 아니다. 자세한 얘기는 [다음 게시글](https://stackoverflow.com/questions/54865746/how-does-k8s-service-route-the-traffic-to-mulitiple-endpoints)에서 확인할 수 있다

<br/>

## **출처**

- https://medium.com/dtevangelist/k8s-kubernetes%EC%9D%98-hpa%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-%EC%98%A4%ED%86%A0%EC%8A%A4%EC%BC%80%EC%9D%BC%EB%A7%81-auto-scaling-2fc6aca61c26
- https://huisam.tistory.com/entry/k8s-hpa
- https://stackoverflow.com/questions/54865746/how-does-k8s-service-route-the-traffic-to-mulitiple-endpoints

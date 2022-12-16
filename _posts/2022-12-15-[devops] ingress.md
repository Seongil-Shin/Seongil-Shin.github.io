---
title: 쿠버네티스 Ingress
author: 신성일
date: 2022-12-15 19:00:00 +0900
categories: [study, devops]
tags: [kubernetes, k8s, ingress]
---

# 쿠버네티스 Ingress

 ingress는 일반적으로 외부로부터 서버 내부로 유입되는 네트워크 트래픽을 뜻한다.

쿠버네티스에서는 ingress라는 리소스 오브젝트가 존재한다. 쿠버네티스의 ingress는 외부에서 쿠버네티스 클러스터 내부로 들어오는 http, https 요청을 어떻게 처리할지 정의한다. 즉, 외부에서 쿠버네티스에서 실행 중인 deployment와 service에 접근하기 위한 일종의 gateway 같은 역할을 담당한다.

ingress를 사용하지 않으면 NodePort나 LoadBalancer를 사용하여 외부 요청을 받을 수 있다. 하지만 `NodePort`와 `LoadBalancer`는 일반적으로 L4(TCP, UDP) 에서의 요청을 처리하며, 네트워크 요청에 대한 세부적인 처리 로직을 구현하기는 한계가 있다.

![kubernetes-loadbalancer-services](https://www.code4projects.net/wp-content/uploads/2020/08/kubernetes-ingress.png)

쿠버네티스에서 ingress를 사용하면 L7에서의 요청을 처리할 수 있다.

- L7 로드 밸런싱
- TLS/SSL 인증서 처리
- path에 따른 라우팅

<br/>

## Ingress와 Ingress Controller

쿠버네티스에서 ingress를 사용하기위해서는 다음 두가지가 필요하다

- `kind: Ingress`로 정의되는 Ingress 오브젝트
- Ingress 규칙이 적용될 Ingress Controller

### Ingress 오브젝트

Ingress를 정의하는 yaml 파일은 다음과 같다.

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: callmeshin75.com   # 1
    http:  # 2
      paths:
      - path: /api/hostname-service # 3
        backend:
          serviceName: hostname-service
          servicePort: 80
```

1. `callmeshin75.com` 호스트명으로 접근하는 네트워크 요청에 대해서 ingress 규칙을 적용
   - host를 지정하지 않으면 지정된 IP 주소를 통해 들어오는 모든 인바운드 HTTP 트래픽에 규칙이 적용된다.
2. http 프로토콜 사용
3. `/api/hostname-service` 경로로 접근하는 요청을 대상으로
4. `hostname-service`라는 서비스의 `80` 포트로 요청을 전달

위 yaml 파일로부터 ingress를 생성해도 아무 일도 일어나지 않는다. ingress는 단지 ingress 규칙을 정의하는 선언적인 오브젝트일 뿐 외부 요청을 받아들이는 실제 서버가 아니기 때문이다. 

Ingress는 `Ingress controller`라고 불리는 서버 컨테이너에 적용되어야만 Ingress에 적용된 규칙이 활성화된다. 즉 Ingress controller는 외부로부터 네트워크 요청을 수신했을 때, Ingress 규칙에 기반해 이 요청을 어떻게 처리할지를 결정한다.

### Ingress controller

![ingress controller](https://www.fairwinds.com/hs-fs/hubfs/Blog%20Feature%20Images/Nginx%20Ingress%20on%20GCP%20-%20Fig%2001.png?width=1794&name=Nginx%20Ingress%20on%20GCP%20-%20Fig%2001.png)

Ingress는 직접 생성해 사용할수도, 클라우드 플랫폼에 위임할 수도 있다. 직접 구동하려면 Nginx 웹 서버 기반의 Nginx Ingress Controller를 사용할 수 있고, 클라우드 플랫폼에 위임하려면 Google Kubernetes Engine을 사용할 수도 있다.

Ingress Controller는 Ingress보다 먼저 생성할 경우, 쿠버네티스로부터 Ingress의 생성을 watch하고 있으므로, Ingress가 생성되면 자동으로 Nginx에 등록된다. 자세한 내용은  [이 포스트](https://www.joyfulbikeshedding.com/blog/2018-03-26-studying-the-kubernetes-ingress-system.html)에 나와있다.

Nginx ingress controller에 관한 자세한 내용은 [공식문서](https://github.com/kubernetes/ingress-nginx/tree/main/charts/ingress-nginx)를 참고하면 된다. (ingress controller 생성에 사용되는 [mandatory.yaml](https://github.com/kubernetes/ingress-nginx/blob/nginx-0.26.1/deploy/static/mandatory.yaml))

## ingress 상태 확인

`kubectl get ingress`로 생성한 ingress의 상태를 확인할 수 있다.

```
NAME           CLASS         HOSTS   ADDRESS         PORTS   AGE
test-ingress   external-lb   *       203.0.113.123   80      59s
```

- HOSTS : ingress 설정에서 세팅한 `host` 필드의 값
- ADDRESS : ingress controller가 ingress를 충족시키기 위해 할당한 IP. 할당될 때까지는 주소는 `pending`으로 표시된다.

<br/>

## 출처

https://kubernetes.io/ko/docs/concepts/services-networking/ingress/

https://blog.naver.com/alice_k106/221502890249
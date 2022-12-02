---
title: Helm
author: 신성일
date: 2022-11-10 18:21:26 +0900
categories: [study, devops]
tags: [devops, helm, k8s, kubernetes]
---

# **Helm**

Kubernetes 패키지 관리를 도와주는 도구. node.js의 npm과 같은 역할을 수행한다.

Helm을 사용하면 쿠버네티스 클러스터에서 동작하도록 작성된 패키지들을 관리할 수 있다. 즉, Helm을 사용하면 클러스터에 배포한 애플리케이션을 쉽게 설치, 업데이트, 삭제할 수 있다.

일반적으로 쿠버네티스는 여러 오브젝트로 구성되어있는데, 만약 애플리케이션을 새로 업데이트하고 싶으면, 애플리케이션을 포함하는 모든 오브젝트마다 `kubectl` 명령을 날려야하는데, Helm을 사용하면 하나의 명령으로 애플리케이션을 배포할 수 있다.

## **주요 개념**

helm은 Chart, Repository, Release 로 구성되어있으며 다음과 같이 연계된다.

> K8s cluster 내부에 Helm chart를 원하는 repository에서 검색후 설치 -> 각 설치에 따른 새로운 Release 생성

각 구성요소는 다음과 같다.

**Chart(차트)**

Helm 패키지로, k8s cluster에서 애플리케이션이 기동되기 위해 필요한 모든 리소스들을 포함. 여러 쿠버네티스 오브젝트를 하나로 묶은 것이라 볼 수 있다.

특정 디렉터리 하위에 모여있는 파일들을 통틀어서 chart라고 부른다. 예시로 WordPress chart는 다음과 같은 구조로 되어있다.

```text
wordpress/
	Chart.yaml               # 차트의 정보를 담고 있음.
	LICENSE                  # OPTIONA
	README.md                # OPTIONAL
	requirements.yaml        # OPTIONAL: 차트의 의존성을 포함한 파일
	values.yaml              # 배포때마다 바뀔 수 있는 값을 포함
	charts/                  # 의존성을 담은 디렉터리
	templates/               # 쿠버네티스 오브젝트 템플릿의 디렉터리. 
													 # 설정값과 결합하여 쿠버네티스 메니페스트를 생성하는데 사용한다.
	templates/NOTES.txt      # OPTIONAL: A plain text file containing short usage notes
```

위와 같은 구조에 version 정보가 붙어서 tgz 형태로 압축 패키징 된다. ex) `nginx-1.2.3.tgz`

**Repository(저장소)**

차트 저장소로, 차트를 모아두고 공유하는 장소

**Release(릴리즈)**

K8s cluster에서 구동되는 차트 인스턴스. 일반적으로 동일한 chart를 여러 번 설치할 수 있고 이는 새로운 Release로 관리되게 된다. Release 될 때 패키지된 차트와 config가 결합되어 정상 실행되게 된다.

<br/>

## Helm 템플릿

애플리케이션을 Helm으로 배포하려면 애플리케이션을 정의하는 Helm 차트를 작성해야한다. Chart 구성에 관한 자세한 내용은 [공식문서](https://v2.helm.sh/docs/developing_charts/)에서 확인 가능하다.

차트를 작성할 때 가장 중요한 부분중 하나는 `template/` 디렉터리이다. 이 디렉터리의 템플릿들이 실제로 생성항 쿠버네티스 리소스를 정의하기 때문이다.

Helm 템플릿은 [Go의 템플릿](https://pkg.go.dev/text/template) 방식을 따른다. `{{}}`로 감까진 영역을 템플릿 디렉티브라고 하는데, 이 템플릿 디렉티브를 활용하여 어떤 값을 특정 값으로 치환하거나 `if/else`와 같은 컨트롤 구조를 나타낼 수 있다. 또한 함수를 사용할 수도 있다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "test-chart.fullname" . }}
  labels:
{{ include "test-chart.labels" . | indent 4 }}
...
{{- if .Values.ingress.enabled -}}
```

[Heml 공식문서의 템플릿 개발 가이드](https://v2.helm.sh/docs/chart_template_guide/)

<br/>

## 출처

- https://tech.ktcloud.com/51
- https://reoim.tistory.com/entry/Kubernetes-Helm-%EC%82%AC%EC%9A%A9%EB%B2%95
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

특정 디렉터리 하위에 모여있는 파일들을 통틀어서 chart라고 부른다.  애플리케이션을 Helm으로 배포하려면 애플리케이션을 정의하는 Helm 차트를 작성해야한다. Chart 구성에 관한 자세한 내용은 [공식문서](https://v2.helm.sh/docs/developing_charts/)에서 확인 가능하며 기본적으로 아래와 같다.

```text
chart/
	Chart.yaml               # 차트의 정보를 담고 있음.
	LICENSE                  # OPTIONA
	README.md                # OPTIONAL
	requirements.yaml        # OPTIONAL: 차트의 의존성을 포함한 파일
	values.yaml              # 배포때마다 바뀔 수 있는 값을 포함
	charts/                  # 애플리케이션의 의존성들의 helm chart를 저장
													 # ex) 애플리케이션 helm chart 설치시 필요하면 mysql helm chart를 저장
	templates/               # 쿠버네티스 오브젝트 템플릿의 디렉터리. 실제 배포시 필요함.
													 # 설정값과 결합하여 쿠버네티스 메니페스트를 생성하는데 사용한다.
	templates/NOTES.txt      # OPTIONAL: A plain text file containing short usage notes
```

위와 같은 구조에 version 정보가 붙어서 tgz 형태로 압축 패키징 된다. ex) `nginx-1.2.3.tgz`

**Repository(저장소)**

차트 저장소로, 차트를 모아두고 공유하는 장소

**Release(릴리즈)**

K8s cluster에서 구동되는 차트 인스턴스. 일반적으로 동일한 chart를 여러 번 설치할 수 있고 이는 새로운 Release로 관리되게 된다. Release 될 때 패키지된 차트와 config가 결합되어 정상 실행되게 된다.

<br/>

## Chart.yaml 

```yaml
apiVersion: v1
appVersion: "1.0"
description: A Helm chart for Kubernetes
name: mychart
version: 0.1.0
```

- apiVersion : helm chart api 버전
  - helm 3 이상이 필요하면 v2 여야함.
  - helm 3 이전을 사용하면 v1을 사용.
- appVersion : 어플리케이션 버전을 나타내는 필드. 정보만 제공하고 차트 버전 계산에 사용되지 않음,
- description : 프로젝트 설명
- name : 차트명
- version : semver 2 버전. 해당 차트의 버전을 나타내는 필드.
- kubeVersion : 호환되는 쿠버네티스 버전의 SemVer 범위 

<br/>

## values.yaml

```yaml
# Default values for mychart.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: nginx
  tag: stable
  pullPolicy: IfNotPresent

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""
```

templates에서 사용할 값들을 저장해놓은 곳. go template

<br/>

## templates

차트를 작성할 때 가장 중요한 부분중 하나는 `templates/` 디렉터리이다. 이 디렉터리의 템플릿들이 실제로 생성할 쿠버네티스 오브젝트를 정의하기 때문이다.

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

[chart 템플릿 작성 가이드](https://helm.sh/ko/docs/chart_template_guide/getting_started/)

### templates/ 디렉터리 규칙

[링크](https://helm.sh/ko/docs/chart_template_guide/named_templates/#%EB%8B%A8%ED%8E%B8partial%EA%B3%BC-_-%ED%8C%8C%EC%9D%BC)

1. `templates/` 아래에 있는 모든 파일은 `NOTES.txt`를 제외하곤 전부 쿠버네티스 메니페스트로 다뤄진다.
2. `_`로 시작하는 파일은 쿠버네티스 메니페스트로 다뤄지지 않는다. 쿠버네티스 오브젝트를 만들지도 않고, 다른 차트 메니페스트에서 사용한다.

<br/>

## Helm 작성법 링크

- [Chart.yaml](https://helm.sh/ko/docs/topics/charts/#chartyaml-%ED%8C%8C%EC%9D%BC)

<br/>

## 출처

- https://tech.ktcloud.com/51
- https://reoim.tistory.com/entry/Kubernetes-Helm-%EC%82%AC%EC%9A%A9%EB%B2%95


---
title: Helm
author: 신성일
date: 2022-11-10 18:21:26 +0900
categories: [study, devops]
tags: [devops, helm, k8s, kubernetes]
---

# **Helm**

Kubernetes 패키지 관리를 도와주는 도구. node.js의 npm과 같은 역할을 수행한다.

## **주요 개념**

helm은 Chart, Repository, Release 로 구성되어있으며 다음과 같이 연계된다.

> K8s cluster 내부에 Helm chart를 원하는 repository에서 검색후 설치 -> 각 설치에 따른 새로운 Release 생성

각 구성요소는 다음과 같다.

**Chart(차트)**

Helm 패키지로, k8s cluster에서 애플리케이션이 기동되기 위해 필요한 모든 리소스들을 포함. 

특정 디렉터리 하위에 모여있는 파일들을 통틀어서 chart라고 부른다. 예시로 WordPress chart는 다음과 같은 구조로 되어있다.

```text
wordpress/
	Chart.yaml               # A YAML file containing information about the chart
	LICENSE                  # OPTIONAL: A plain text file containing the license for the chart
	README.md                # OPTIONAL: A human-readable README file
	requirements.yaml        # OPTIONAL: A YAML file listing dependencies for the chart
	values.yaml              # The default configuration values for this chart
	charts/                  # A directory containing any charts upon which this chart depends.
	templates/               # A directory of templates that, when combined with values,
													 # will generate valid Kubernetes manifest files.
	templates/NOTES.txt      # OPTIONAL: A plain text file containing short usage notes
```

위와 같은 구조에 version 정보가 붙어서 tgz 형태로 압축 패키징 된다. ex) `nginx-1.2.3.tgz`

**Repository(저장소)**

차트 저장소로, 차트를 모아두고 공유하는 장소

**Release(릴리즈)**

K8s cluster에서 구동되는 차트 인스턴스. 일반적으로 동일한 chart를 여러 번 설치할 수 있고 이는 새로운 Release로 관리되게 된다. Release 될 때 패키지된 차트와 config가 결합되어 정상 실행되게 된다.



## 출처

- https://tech.ktcloud.com/51
- https://reoim.tistory.com/entry/Kubernetes-Helm-%EC%82%AC%EC%9A%A9%EB%B2%95
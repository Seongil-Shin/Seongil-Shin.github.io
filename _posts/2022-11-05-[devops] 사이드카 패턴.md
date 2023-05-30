---
title: 사이드카 패턴
author: 신성일
date: 2022-11-05 18:20:26 +0900
categories: [study, devops]
tags: [sidecar]
---

# 사이드카 패턴

쿠버네티스의 패턴 중 하나로, 어플리케이션 컨테이너와 독립적으로 동작하는 별도의 컨테이너를 붙이는 패턴이다. 어플리케이션 컨테이너의 변경이나 수정 없이 독립적으로 동작하는 컨테이너를 붙였다 뗐다 할 수 있다.

<br/>

## 파드에서의 사이드카

파드는 쿠버네티스에서 가장 기본적인 배포 단위로서 자신에게 속한 컨테이너들에게 런타임제약을 걸 수 있다. 예를 들어 모든 컨테이너는 동일한 노드에 배치되고, 동일한 파드 수명주기를 공유하게 할 수 있다. 파드는 또한 컨테이너들이 볼륨을 공유하고 로컬 네트워크 또는 호스트 IPC를 통해 서로 통신할 수 있게 해준다. 이러한 이유로 컨테이너 그룹을 파드로 만든다.

사이드카 패턴은 컨테이너의 기능을 확장시키기 위해 컨테이너를 파드에 넣는 것과 유사하다. 다음은 HTTP 서버와 깃 동기화 컨테이너가 정의된 파일이다. HTTP가 어플리케이션 컨테이너이고, 깃 동기화 컨테이너가 사이드카 컨테이너이다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  containers:
    - name: app
      image: docker.io/centos/httpd // HTTP로 파일을 제공하는 기본 애플리케이션 컨테이너
      ports:
        - containerPort: 80
      volumeMounts:
        - mountPath: /var/www/html  //사이드카와 기본 애플리케이션 컨테이너 간에 데이터를 교환하기위해 공유된 장소
          name: git
    - name: poll
      image: axeclbr/git            // 깃 서버로부터 데이터를 가져오고 병렬로 실행하는 사이드카 컨테이너
      volumeMounts:
        - mountPath: /var/lib/data //사이드카와 기본 애플리케이션 컨테이너 간에 데이터를 교환하기 위해 공유된 장소
          name: git
      env:
      - name: GIT_REPO
        value: https://github.com/mdn/beginner-html-site-scripted
      command:
        - "sh"
        - "-c"
        - "git clone ${GIT_REPO} . && watch -n 600 git pull"
        workingDir: /var/lib/data
      volumes:
        - emptyDir: {}
          name: git
```

사이드카 패턴에서는 기본 컨테이너가 있고, 공동작업을 향상시키는 보조 컨테이너가 있다. 일반적으로 기본 컨테이너는 컨테이너 목록에 나열된 첫번째 컨테이너가 기본 컨테이너다.

사이드카 패턴을 통해 컨테이너는 런타임에 협업이 가능해지며 그와 동시에 두 컨테이너의 관심을 분리시킴으로서 서로의 의존성을 없앤다.

또한 사이드카는 같은 파드 안에서 애플리케이션 컨테이너와 동일한 수명주기를 공유하므로 애플리케이션 컨테이너와 함께 만들어지고 사용중지된다.

<br/>

## 예시

-  보안을 위해 사이드카로 Nginx reverse proxy를 붙여 https 통신을 한다.
-  성능을 위해 사이드카로 Nginx content cache 등을 붙인다.
-  인프라 개발팀은 인프라에 액세스할 언어별 라이브러리가 아닌 각 애플리케이션과 함께 배포될 서비스를 만든다. 인프라 서비스는 사이드카로 로드되고, 로깅/검색 등 인프라 서비스에 대한 공통 계층을 제공한다. 또한 애플리케이션 컨테이너의 호스트 환경 및 프로세스를 모니터링하고 중앙집중식 서비스에 정보를 기록한다.

## 장단점

장점

-  어플리케이션 간의 상호의존성을 줄일 수 있다.
-  대부분 같은 스토리지를 공유할 수 있기 때문에 공유에 대한 고민이 적다

단점

-  어플리케이션이 너무 작은 경우 배보다 배꼽이 더 커진다.
-  프로세스간 통신이 많기에 최적화해야한다면, 한 어플리케이션에서 함께 처리하는게 좋을 수 있다.

[참고문서](https://blog.leocat.kr/notes/2019/02/16/cloud-sidecar-pattern)

[참고문서2](https://velog.io/@youngerjesus/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%ED%8C%A8%ED%84%B4-%EC%82%AC%EC%9D%B4%EB%93%9C%EC%B9%B4)

[참고문서3](https://azderica.github.io/00-design-pattern-sidecar/)

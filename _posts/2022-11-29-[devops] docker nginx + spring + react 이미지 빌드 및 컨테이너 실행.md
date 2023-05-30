---
title: nginx + spring boot+ react로 구성된 앱 dockerfile 작성
author: 신성일
date: 2022-11-29 18:19:26 +0900
categories: [study, devops]
tags: []
---

# nginx + spring boot+ react로 구성된 앱 dockerfile 작성

## docker flow

![docker flow](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/docker-application-development-process/media/docker-app-development-workflow/life-cycle-containerized-apps-docker-cli.png)

도커에서 이미지를 빌드하고, 앱이 실행되는 컨테이너를 실행하는 과정은 크게 `이미지 빌드`, `이미지 푸시`, `컨테이너 실행` 단계로 나뉜다. nginx + spring boot + react로 구성된 앱을 도커 이미지로 관리하려면 먼저 이미지를 빌드할 dockerfile을 잘 작성해야한다. 

먼저 이번 글에서 사용될 프로젝트(빌드 컨텍스트)의 디렉터리 구조부터 알아보자

<br/>

## 프로젝트 구조

```
.
├── Dockerfile
├── nginx.conf
└── startApp.sh
├── client
│   ├── README.md
│   ├── build
│   ├── node_modules
│   ├── package-lock.json
│   ├── package.json
│   ├── patches
│   ├── postcss.config.js
│   ├── public
│   ├── src
│   ├── tailwind.config.js
│   ├── tsconfig.json
├── server
│   ├── bin
│   ├── build
│   ├── build.gradle
│   ├── gradle
│   ├── gradlew
│   ├── gradlew.bat
│   ├── logs
│   ├── out
│   ├── settings.gradle
└── ├── src
```

최상단 디렉터리에 `Dockerfile`과 nginx 설정에 필요한 `nginx.conf`를 배치해두었다. `startApp.sh`는 컨테이너가 실행될 때 스프링과 nginx를 실행할 스크립트 파일이다. React와 Spring boot는 각각 client, server 디렉터리에 배치하였다.

이제 본격적으로 dockerfile을 작성해보자. 이때 dockerfile은 multi stage build로 작성하였고, spring boot 빌드 -> react 빌드 -> nginx 및 spring 설정 및 실행 과정을 거치도록 작성하였다.

<br/>

## Spring Boot 빌드

```dockerfile
# syntax=docker/dockerfile:1.2
# Stage 1 - spring 빌드
FROM openjdk:18-jdk-alpine AS spring-builder
COPY ./server/gradlew ./
COPY ./server/gradle ./gradle
COPY ./server/settings.gradle ./
COPY ./server/build.gradle ./
COPY ./server/src ./src
RUN ./gradlew bootJar
```

- FROM : spring boot 빌드를 위해 `openjdk`를 base image로 삼았다. 
- COPY : 빌드에 필요한 파일들을 빌드 컨텍스트에서 베이스이미지로 복사한다
- RUN : 빌드 명령어를 실행하여 spring을 빌드한다.

### gradlew 실행시 권한 오류가 발생했을 경우

두가지 해결책이 있다.

- 빌드 컨텍스트, 즉 내 로컬에 있는 gradlew 파일의 권한을 바꾼다.

- 베이스 이미지 상에서 gradlew 파일 권한을 바꾼다. 

  다음과 같이 RUN 하기 전에 바꾸면 됨.

  ```dockerfile
  RUN chmod +x ./gradlew
  RUN ./gradlew bootJar
  ```

<br/>

## React 빌드

```dockerfile
# Stage 2 - react 빌드
FROM node:alpine AS react-builder
WORKDIR /usr/src/app
# 의존성 설치를 위한 파일 복사
COPY ./client/package-lock.json ./
COPY ./client/package.json ./
RUN npm ci # 의존성 설치
# React 빌드를 위한 파일 복사
COPY ./client/tailwind.config.js ./tailwind.config.js
COPY ./client/postcss.config.js ./postcss.config.js
COPY ./client/tsconfig.json ./
COPY ./client/public ./public
COPY ./client/src ./src
RUN npm run build
```

- FROM : react 빌드를 위해 node를 baseimage로 삼았다
- WORKDIR : 작업할 디렉터리 지정
- COPY ~ : React 빌드를 위한 파일 및 npm 의존성 설치를 위한 파일을 base image로 복사한다.
- RUN : 의존성을 설치한다. 이때 `npm ci`를 사용하는 것이 package-lock.json을 지키며 의존성을 설치할 수 있어서 좋다. 버전에 큰 문제가 없다면 `npm install`을 사용해도 된다
- COPY ~ : 소스 코드를 가져온다.
- RUN : react를 빌드한다.

### COPY, RUN 배치 전략

React 빌드를 위한 dockerfile을 봤을 때, COPY, RUN 커맨드를 어떤 기준으로 배치하였는지 궁금할 수 있다. 
docker에서는 이미지는 여러 레이어로 구성되어있다. `FROM`,  `RUN`, `COPY`, `ADD` 등의 커맨드를 기준으로 레이어를 구성한다. 레이어는 이미지 빌드 시간을 최소화하기 위한 것으로, 이미지를 빌드할 때 변경되지 않은 레이어는 기존에 저장된 데이터를 사용하고, 변경된 레이어는 그 레이어부터 상위 레이어까지 새로 명령어를 실행한다.

![docker layer](https://subicura.com/assets/article_images/2017-01-19-docker-guide-for-beginners-1/image-layer.png)

따라서 COPY, RUN 커맨드를 사용할 땐, 커맨드의 결과물이 자주 변경되지 않는 것은 가능한 상단에 두어 낮은 레이어에 위치시키면 이미지 빌드 속도를 크게 개선할 수 있다.

따라서 위 react 빌드 dockerfile에서는 상대적으로 자주 변경되지않는 package.json 과 같은 의존성 관리 파일을 상단에 두어 먼저 설치하고, 상대적으로 자주 변경되는 소스파일을 복사하는 COPY 명령어를 후순위에 둔 것이다.

이때 `RUN`, `COPY`, `ADD` 커맨드는 도커 이미지 사이즈에 영향을 주는 커맨드들이다. **잘못 쓰일 수록 도커의 이미지가 필요이상으로 커지므로** 가능한 적절히 사용하는 것이 좋다.

참고로 도커 레이어 캐시가 저장되는 곳은 OS마다 다르지만, 기본적으로는 사용자 로컬 환경이다. 이미지 빌드시에 `--graph` 옵션을 주어 레이어 캐시가 저장되는 위치를 변경할 수 있다.

<br/>

## nginx 및 spring 준비

```dockerfile
# Stage 3
FROM amd64/nginx

# jdk 설치
WORKDIR /home1/irteam/download
RUN curl -X GET https://download.oracle.com/java/17/latest/jdk-17_linux-x64_bin.tar.gz --output jdk-17.tar.gz && \
    tar xvzf jdk-17.tar.gz && \
    rm jdk-17.tar.gz

# 어플리케이션 준비
COPY ./nginx.conf /etc/nginx/nginx.conf
COPY --from=spring-builder /build/libs/*.jar /app/server/spring-server.jar
COPY --from=react-builder /usr/src/app/client/build /app/client/build
```

- FROM : `amp64/nginx` 를 base image로 삼는다.
- RUN curl ~ : spring 실행에 필요한 jdk를 다운로드 받고, 압축해제한다
  - `RUN` 커맨드는 레이어를 생성하고, 도커 이미지 사이즈에 영향을 준다. 만약 `curl`로 `jdk`파일을 가져오는 작업과, 압축해제, 삭제 작업을 분리하여 `RUN`으로 실행하면 어떻게 될까?
  - 하나하나가 개별의 레이어로 저장되고, 최종적으로 필요없는 압축 해제 전의 `jdk` 파일마저 이미지에 포함되게 된다.
  - 따라서 최대한 이미지 사이즈를 줄이기 위해서는 `&&` 을 사용하여 한줄에 명령어를 모두 작성하였다.

- COPY : nginx.conf 파일을 가져와 nginx 설정을 하고, 어플리케이션 실행을 위한 파일을 이전 빌더로부터 가져온다.

<br/>

## 실행

```dockerfile
# 실행 스크립트
COPY ./startApp.sh ./
ENTRYPOINT ["./startApp.sh"]
```

- 어플리케이션 실행을 위한 실행 파일을 가져오고, ENTRYPOINT로 커맨드를 작성하여 컨테이너가 실행하자마자 실행 스크립트가 실행시킨다.
- 실행 스크립트는 아래와 같다.

```shell
#!/bin/sh

nginx
/jdk-17.0.5/bin/java -jar /app/server/spring-server.jar
```

<br/>

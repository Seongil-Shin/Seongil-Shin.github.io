---
title: next.js 최적화과정
author: 신성일
date: 2023-10-22  16:00:00 +0900
categories: [study, next.js]
tags: [next.js]
---

<!-- @format -->

## 발단

서비스를 운영하던 도중 CPU/Memory 사용량은 낮은데 사용자 응답이 매우 느려지는 상황이 발생하였다.
서비스는 쿠버네티스로 띄워져있었고, HPA metric으로 CPU와 memory가 걸려있었는데, CPU와 memory가 올라가지 않으니 scale-out도 발생하지 않아 적은 pod으로 계속 서비스 되고 있었다.
수동으로 pod을 3~4배 가량 늘려 대응은 하였으나 추후에도 발생할 이슈일 것으로 보여 대응을 하기로 했다.

## 테스트

이 이슈는 next.js로 띄워져있는 페이지 중 하나에 페이지에 갑자기 트래픽이 몰려 발생한 이슈였다.
따라서 당시 리얼환경과 비슷하게 환경 세팅을 하고 [ngrinder](https://naver.github.io/ngrinder/)를 사용하여 대규모 트래픽 테스트를 하였다.
이슈가 발생했던 페이지를 포함하여 다양한 페이지를 테스트한 결과는 다음과 같았다. (TPS가 낮음 -> 성능이 낮음)

-   페이지 A (이슈가 발생했던 페이지) : TPS 매우 낮음.
-   페이지 B : TPS 낮음
-   페이지 C : TPS 중간
-   페이지 D : TPS 높음
-   페이지 E : TPS 매우 높음

해당 페이지들은 다음과 같은 특징을 가지고 있다.

-   페이지 A (TPS 매우 낮음.)
    -   동적 페이지
    -   API 소스 A'로의 요청 2개, 소스 B'로의 요청 1개
    -   API 소스 A'는 C'로부터 데이터를 중개해주는 역할을 함.
-   페이지 B (TPS 낮음)
    -   동적페이지
    -   API 소스 B'로의 요청 1개
-   페이지 C (TPS 중간)
    -   동적페이지
    -   API 소스 A'로의 요청 2개
    -   이 A'로 간 API 두개는 A'가 DB로부터 바로 데이터를 가져옴.
-   페이지 D (TPS 높음)
    -   동적페이지
    -   API 요청 없음
-   페이지 E (TPS 매우 높음)
    -   정적페이지

이 결과들을 보고 내린 결론은 다음과 같다.

-   성능 저하가 크지 않은 경우
    -   next.js 내에서만 연산이 이루어지는 경우 (페이지 D, E)
    -   A'로 보내는 API 중 A'가 스스로 처리하는 API를 사용하는 경우 (페이지 C)
-   성능 저하가 큰 경우
    -   A'로 보내는 API 중 A'가 중개 API 로써 동작하는 경우 (페이지 A)
    -   B'로 API를 요청하는 경우 (페이지 A, B)

## 최적화

아래 두 전략을 가지고 최적화를 진행하였다.

1. API 요청을 최소화하기
2. 요청량을 기준으로 HPA metric을 산정하기

자세한 내용은 다음과 같다.

### API 요청 최소화하기

기본적으로 API 요청이 발생하는 페이지에서 성능저하가 발생하였다. 따라서 API 요청을 최소화하는게 최우선 목표라고 생각하였다.

#### 불필요한 호출 줄이기

가장 성능이 안좋았던 페이지 A는 A'로 API 요청을 2개 보내고, B'로 1개의 요청을 보낸다.
A'로 보내는 두개의 요청은 기존에는 다음과 같이 코드가 짜여있었다.

```js
export const getServerSideProps = async(context) => {
	...
	const userProfileData = await getUserProfileData();
	const userContentData = await getUserContentData();
	...
	return {
		props : {
			userData,
			userContentData,
		}
	}
}
```

`getUserProfileData()` 와 `getUserContentData()`는 유저가 로그인 되어있든 안되어있든 항상 A'로 요청을 보낸다. 그리고 A'는 로그인 되어있지 않으면 null을 응답한다.
처음 이 코드를 짰을때는 성능에 대해 크게 걱정이 없어서 이렇게 항상 요청하는 식으로 짰다.(코드 조금 더 쓰기 귀찮았다..) 하지만 로그인 여부는 next.js 단에서도 API 요청 없이 알 수 있도록 설정이 되어있었다. 따라서 다음과 같이 next.js 에서 먼저 로그인 여부를 확인하고 로그인 되어있으면 요청하는 식으로 코드를 변경하였다.

```js
export const getServerSideProps = async(context) => {
	const { isLoggedIn } = getAuthentication();
	...
	let userProfileData = null, userContentData = null;
	if(isLoggedIn) {
		userProfileData = await getUserProfileData();
		userContentData = await getUserContentData();

	}
	...
	return {
		props : {
			userData,
			userContentData,
		}
	}
}
```

이로써 미로그인 유저들이 불필요한 API 요청을 보내는 일이 없어졌다. 조금 더 개선을 하자면, `getUserProfileData()` 와 `getUserContentData()`를 하나의 API로 합칠 수도 있으나 이건 API 개발자의 작업이니 이 글에선 자세히 작성하진 않겠다.

또 여기서 한번 더 최적화를 한다면 다음과 같이 `Promise.all`을 사용하여 요청을 동시에 보낼 수도 있다.

```js
export const getServerSideProps = async(context) => {
	const { isLoggedIn } = getAuthentication();
	...
	let userProfileData = null, userContentData = null;
	if(isLoggedIn) {
		[userProfileData, userContentData] = await Promise.all([getUserProfileData(), getUserContentData()])
	}
	...
	return {
		props : {
			userData,
			userContentData,
		}
	}
}
```

기존에는 `getUserProfileData()`의 응답시간 + `getUserContentData()`의 응답시간으로 최종 응답시간이 정해졌으나, `Promise.all`을 사용하면 둘 중 더 느린 응답시간으로 최종 응답시간이 결정되기 때문이다.

#### 캐시 사용

다음은 B'로 보내는 요청을 최적화한 방법이다. B'는 콘텐츠를 가져오는 API로 기존에는 매요청마다 호출하고 있었다.

```js
export const getServerSideProps = async(context) => {
	...
	const contentData = await getContentData('page-a');
	...
	return {
		props : {
			contentData
		}
	}
}
```

콘텐츠의 내용이 변경가능하여 항상 호출하였으나, 변경 사항을 즉시 반영해야할 심각도를 가진 콘텐츠는 아니었다. 즉 캐시를 사용하여 콘텐츠 변경의 반영이 어느정도 늦어져도 문제는 없었다. 따라서 [memory-cache](https://www.npmjs.com/package/memory-cache)라는 in-memory cache 을 쉽게 사용할 수 있게 도와주는 라이브러리를 사용하기로 하였다.

```js
import cacheData from "memory-cache";

export const getServerSideProps = async(context) => {
	...
	const key = "page-a"
	let contentData = cacheData.get(key);
	if (!contentData) {
		contentData = await getContentData(key);
		cacheData.put(key, contentData, 60000);
	}
	...
	return {
		props : {
			contentData
		}
	}

}
```

이렇게 두가지만 적용한 후, 다시 페이지 A를 ngrinder로 테스트한 결과는 다음과 같았다.

-   미로그인 유저 -> TPS 15배 상승
-   로그인 유저 -> TPS 2~3배 상승
    로그인 유저의 경우 미로그인 유저에 비해 상승량은 적었지만 기존보다는 크게 상승했다.

### HPA metric을 요청량 기준으로 변경

이번 이슈를 겪으면서 알게 된 것은 CPU, Memory 사용량이 많아져서 응답이 느리게 갈 가능성은 적다는 것이었다. 정적페이지 및 API 요청 없는 동적페이지의 경우 매우 빠르게 처리하고 있었고, pod 개수도 많아서 예상되는 최대치의 트래픽이 들어오더라도 CPU, Memory 사용량이 많이 점유되는 가능성은 적었다. 반대로 API 요청이 발생하는 페이지에서는 예상보다 훨씬 적은 트래픽이 들어오더라도 터지는 일이 발생하였다.

따라서 서버 응답 장애가 발생한다면, CPU/Memory 사용량은 낮지만 API 요청이 발생하는 페이지에 요청이 몰려 발생할 확률이 훨씬 컸다.

이에따라 기존에는 HPA metric을 다음과 같이 CPU/Memory 기준으로 작성하였는데, network 요청량이 일정 수준이상이 되면 발생하도록 변경하기로 하였다. (아래 yaml 파일들은 실제 파일이 아닌 예시)

```yaml
# 기존 hpa 설정
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
    name: next-app-hpa
    namespace: default
spec:
    scaleTargetRef:
        apiVersion: apps/v1
        kind: Deployment
        name: next-app
    minReplicas: 1
    maxReplicas: 10
    targetCPUUtilizationPercentage: 50 # CPU가 50%로 맞춰지도록 스케일링 발생
    targetmemoryutilizationpercentage: 50 # Memory가 50%로 맞춰지도록 스케일링 발생
```

```yaml
# 변경한 hpa 설정
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
    name: next-app-hpa
    namespace: default
spec:
    scaleTargetRef:
        apiVersion: apps/v1
        kind: Deployment
        name: next-app
    minReplicas: 1
    maxReplicas: 10
    metrics:
        - object:
              describedObject:
                  apiVersion: v1
                  kind: Service
                  name: test
              metric:
                  name: test_requests_per_second
              target:
                  type: Value
                  value: 200 # 초당 200번 이상 요청이 들어올 시 scale-out 발생
          type: Object
        - resource:
              name: cpu # CPU 설정도 혹시 모르니 남겨둠(CPU가 터지는 상항이 생길 수 있으니..)
              target:
                  averageUtilization: 50
                  type: Utilization
          type: Resource
```

[참고](https://tech.scatterlab.co.kr/kubernetes-hpa-custom-metric/)

다시 ngrinder로 테스트를 하니 scale-out 이 정상적으로 동작하였다.
이 두가지 방법을 적용하여 최적화를 완료하였고, 이후 트래픽이 많이 발생하여도 정상적으로 대응 가능하였다.

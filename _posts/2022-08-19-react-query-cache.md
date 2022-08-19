---
title : React-query 에서 cache를 사용하는 방법
author : 신성일
date : 2022-08-19 21:00:00 +0900
categories : [study, react]
tags: [react, react-query, cache]
---



## useQuery 설정

React-Query 공식문서상 캐싱개념은 stale과 cachetime을 통해 이루어진다

useQuery의 옵션으로 `staletime`과 `cachetime`을 보낼 수 있다.

- staletime : fetch를 통해 전달받은 데이터는 리액트 쿼리의 자료구조 내용중 캐시에 저장되는데, 이 캐시데이터의 '신선한 상태'가 언제까지 될지를 말해주는 옵션
  - default = 0, 받아오는 즉시 stale하다고 판단하여 캐싱데이터와 무관하게 계속해서 fetching을 날림
- cachetime : 캐시 구조에 저잘된 데이터는 메모리상에 존재하게 된다. 이때 메모리에 저장되어있는 캐시데이터가 언제까지 유지될지 말해주는 옵션. 

<br/>

## QueryClient

캐싱과 관련된 내용을 담고있는 객체. useQueryClient 로 접근 가능하다.

사용하기 위해선 먼저 다음과 같이 `<QueryClientProvider>` 로 하위 컴포넌트를 감싸주어야한다. (React Query 자체를 사용하기 위해서도 감싸주어야한다)

```react
const queryClient = new QueryClient({
    defaultOptions : {
        queries : {
            refetchOnWindowFocus:false,
            refetchOnMount:false,
            retry:false
        }
    }
})

return (
	<QueryClientProvider client={queryClient}>
    	<Main>
        	...
        </Main>
    </QueryClientProvider>
)
```

이후 아래와 같이 useQueryClient로 queryClient 객체를 가져오면 다음과 같이 queriesMap 형태로 캐시된 데이터를 확인할 수 있다.

```react
const queryClient = useQueryClient();
console.log(queryClient);
```

![img](https://user-images.githubusercontent.com/58500558/157851602-c0e2219b-0ecd-4649-a7b2-2b399a8a4e43.png)

사진에서 보이듯이 `queriesMap`에서는 useQuery에서 사용한 키값으로 캐싱되어있음을 알 수 있다. useQuery를 사용할때, `staletime`과 `cachetime`을 지정해주면, 그 설정값에 맞추어 키값에 맞는 캐싱된 데이터를 자동으로 가져온다. 이때 요청은 보내지 않는다.

> staletime이 0이라면, 항상 데이터가 stale하다고 여겨지기에 항상 refetching을 진행한다.

<br/>

## 캐싱 구현법

**enabled:false**

useQuery의 옵션으로 enabled:false를 설정해놓으면, 초기 마운트시에 해당 useQuery가 마치 useEffect처럼 첫 마운트시 fetcher 함수 호출을 하고, 실패했을 때는 계속 retry하는 행위를 사전 차단할 수 있다.

그러나 enabled:false는 useQuery 기능을 사용하지 않겠다고 말해주는 것과 같기 때문에, 수동적으로 호출해줘야한다. 그것이 바로 useQuery 함수 호출을 하여 리턴되는 객체에 포함되어있는 refetch 함수다.

refetch함수는 캐싱 데이터를 조회하지 않고 요청을 날린다.

따라서 **초기 마운트시에는 캐싱된 데이터를 가져오되, 원할때는 새로운 데이터를 가져오고 싶을 때**는 enabled:false와 refetch 함수를 사용하면 된다.

<br/>

**enabled:true**

다음과 같이 enabled 옵션을 특정한 상태일때만 true로 만들고, 그 외에는 false로 하여 초기 요청을 통한 retry로 오류를 생성하는 것을 막는다.

```react
const {data, refetch, ...rest} = useQuery([page, "search"], ..., {
                                          enabled : searchValue !== ''
                                          })
```

그 뒤, 조건부로 enabled가 true로 변경되면서 요청을 날리게 되어 성공하면 data 프로퍼티에 그 값이 저장되고, 캐싱에도 저장될 것이다.

<br/>



https://darrengwon.tistory.com/1517

key가 null or undefined일때는?

## 참고

useQuery의 옵션에 사용하는 `onError`, `onSuccess`, `onSettled` 등의 ajax 요청 결과로 동작하는 이벤트는 캐싱값을 가져올때는 동작하지 않는다.

따라서 캐싱값을 사용할때는, 해당 데이터의 결과값은 변동된다는 사실을 이용하여 이 데이터를 이용한 조건부렌더링을 사용하여야한다.

<br/>

## 출처

https://velog.io/@chltjdrhd777/React-Query-%EC%BA%90%EC%8B%B1%EC%97%90-%EB%8C%80%ED%95%9C-%EA%B5%AC%ED%98%84
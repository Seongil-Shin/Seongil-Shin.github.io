---
title: RestTemplate
author: 신성일
date: 2022-11-12 20:19:26 +0900
categories: [study, spring]
tags: []
---


스프링에서 지원하는 객체로, 간편하게 Rest 방식 API를 호출할 수 있는 스프링 내장 클래스다.

스프링 3.0부터 지원하였고, json, xml 응답을 모두 받을 수 있다.

## **특징**

- Blocking I/O 기반의 동기방식을 사용하는 템플릿
- RESTful 형식에 맞추어진 템플릿
- Header, Content-Type 등을 설정하여 외부 API 호출
- Server to Server 통신에 사용

## **동작원리**

![RestTemplate](https://t1.daumcdn.net/cfile/tistory/99300D335A9400A52C)

1. 애플리케이션 내부에서 REST API에 요청하기 위해 RestTemplate의 메서드를 호출한다.
2. RestTemplate은 MessageConverter를 이용해 java object를 request body에 담을 message(JSON etc.)로 변환한다. 메시지 형태는 상황에 따라 다름
3. ClientHttpRequestFactory에서 ClientHttpRequest을 받아와 요청을 전달한다.
4. 실질적으로 ClientHttpRequest가 HTTP 통신으로 요청을 수행한다.
5. RestTemplate이 에러핸들링을 한다.
6. ClientHttpResponse에서 응답 데이터를 가져와 오류가 있으면 처리한다.
7. MessageConverter를 이용해 response body의 message를 java object로 변환한다.
8. 결과를 애플리케이션에 돌려준다.

<br/>

## **API 클래스/메서드 종류**

**클래스**

- RestTemplate : Spring 3부터 지원하였고, API 호출 이후 응답을 받을 때까지 기다리는 동기방식
- AsyncRestTemplate : spring 4에 추가된 비동기 RestTemplate. spring 5에서는 deprecated 됨.
- WebClient : spring 5에서 추가된 논블럭, 리액티브 웹 클라이언트로 동기, 비동기 방식을 지원함

**메서드**

![images%2Fsoosungp33%2Fpost%2Fd02efacc-56e6-4abc-a390-12ecbb565c6b%2Fimage](https://velog.velcdn.com/images%2Fsoosungp33%2Fpost%2Fd02efacc-56e6-4abc-a390-12ecbb565c6b%2Fimage.png)

<br/>

## 사용예시

```java
@Component
public class sendData {
    private static RestTemplate restTemplate;

    public static ResponseEntity<SubmitData> sendEngine() {
        int SubNum = 123;
        int Pnum = 1;
        Object Pcode = "코드";
        SubmitData requestDto = SubmitData.builder()
                .SubNum(SubNum)
                .Pnum(Pnum)
                .Pcode(Pcode)
                .build();

        HttpHeaders headers = new HttpHeaders();
        headers.add("Content-Type", "application/json");
        HttpEntity<SubmitData> entity = new HttpEntity<>(requestDto, headers);

        String url = "http://localhost:8080/send";
        
        return restTemplate.exchange(url, HttpMethod.POST, entity, SubmitData.class);
    }
}
```

<br/>

## **출처**

https://blog.naver.com/hj_kim97/222295259904

https://velog.io/@soosungp33/%EC%8A%A4%ED%94%84%EB%A7%81-RestTemplate-%EC%A0%95%EB%A6%AC%EC%9A%94%EC%B2%AD-%ED%95%A8
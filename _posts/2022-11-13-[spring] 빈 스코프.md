---
title: 빈 스코프
author: 신성일
date: 2022-11-13 21:19:26 +0900
categories: [study, spring]
tags: []
---



빈 스코프는 말 그대로 빈이 존재할 수 있는 범위를 뜻한다. 스프링에서는 다음과 같은 스코프를 지원한다

- 싱글톤 : 기본 스코프. 스프링 컨테이너의 시작과 종료까지 유지되는 가장 넓은 범위의 스코프
- 프로토타입 : 스프링 컨테이너는 프로토타입 빈의 생성과 의존관계 주입까지만 관여하고 더는 관리하지 않는 매우 짧은 범위의 스코프이다.
- 웹 관련 스코프
  - request : 웹 요청이 들어오고 나갈때까지 유지되는 스코프
  - session : 웹 세션이 생성되고 종료될 때까지 유지되는 스코프
  - application : 웹의 서블릿 컨텍스트와 같은 범위로 유지되는 스코프

<br/>

## 프로토타입 스코프

싱글톤 스코프의 빈을 조회하면 스프링 컨테이너는 항상 같은 인스턴스의 스프링 빈을 반환한다. 반면에 프로토타입 스코프를 스프링 컨테이너에 조회하면 스프링 컨테이너는 항상 새로운 인스턴스를 생성해서 반환한다.

![](https://img1.daumcdn.net/thumb/R800x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcVAvFq%2FbtrBZyT4fB2%2FkLzkc637pYEb025QAXtWYK%2Fimg.png)

1. 프로토타입 스코프의 빈을 스프링 컨테이너에 요청한다.
2. 스프링 컨테이너는 이 시점에 프로토타입 빈을 생성하고, 필요한 의존관계를 주입한다.

![](https://velog.velcdn.com/images/shinyeongwoon/post/f3e969f6-4649-4060-aa3f-ac2943f15c12/image.png)

3. 스프링 컨테이너는 생성한 프로토타입 빈을 클라이언트에 반환한다.
4. 이후에 스프링 컨테이너에 같은 요청이 오면 항상 새로운 프로토타입 빈을 생성해서 반환한다.

스프링 컨테이너는 프로토타입 빈을 관리하지 않는다. 프로토타입 빈을 관리할 책임은 프로토타입 빈을 받은 클라이언트에게 있다. 따라서 `@PreDestroy` 같은 종료 메서드가 호출되지 않는다.

**싱글톤 빈과 함께 사용시 문제점**

프로토타입 빈은 매번 새로운 객체 인스턴스를 생성하다는 점에서 싱글톤 빈과 함께 사용시 의도한 대로 동작하지 않을 수 있다.

만약 싱글톤으로 관리되는 빈에서 프로토타입 빈을 의존한다면, 싱글톤 빈이 생성되는 시점에 프로토타입 빈은 단 한번 주입된다. 따라서 클라이언트 요청이 얼마나 많든 싱글톤 빈에 주입된 프로토타입 빈은 새로 생성되지 않는다. 

만약 프로토타입 빈은 매 요청마다 새롭게 생성하는 것을 의도하여 사용되는데, 이렇게 된다면 의도한대로 동작하는 것이 아닐 것이다.

> 참고 : 여러 빈에서 같은 프로토타입 빈을 주입 받으면, **주입 받는 시점에 각각 새로운 프로토타입 빈이 생성**된다. 예를 들어서 clientA, clientB가 각각 의존관계 주입을 받으면 각각 다른 인스턴스의 프로토타입 빈을 주입 받는다.
>
> - clientA prototypeBean@x01
>
> - clientB prototypeBean@x02
>
>   물론 사용할 때 마다 새로 생성되는 것은 아니다

**싱글톤 빈과 함께 사용시 Provider로 문제 해결**

싱글톤 빈과 프로토타입 빈을 함께 사용할 때, 사용할 때마다 항상 새로운 프로토타입 빈을 생성하기 위해선 DI가 아닌 직접 필요한 의존관계를 찾는 것을 의미하는 `Dependency Lookup(DL)`이 필요하다. DL을 위해선 다음과 같은 방법들이 있다.

- 스프링 컨테이너에 요청 : ApplicationContext 클래스로 사용할 때마다 프로토타입 빈 직접 가져오기

- ObjectFactory, ObjectProvider

  ```java
  @Autowired
  private ObjectProvider<PrototypeBean> prototypeBeanProvider;
  
  public int logic() {
        PrototypeBean prototypeBean = prototypeBeanProvider.getObject();
        prototypeBean.addCount();
        int count = prototypeBean.getCount();
        return count;
  }
  ```

  - 둘 다 스프링에 의존하는 클래스로, ObjectFactory은 과거 버전이고, ObjectProvider은 ObjectFactory에 편의기능이 더해진 클래스이다.
  - ObjectProvider은 `getObject()`를 호출하면 내부에서 스프링 컨테이너를 통해 해당 빈을 찾아서 반환한다.
  - 스프링컨테이너에 직접 요청하는 것보다 단위테스트를 만들거나 mock 코드를 만드는 것이 쉬워진다.

- JSR-330 Provider

  - `javax.inject.Provider` 이라는 JSR-330 자바표준이다
  - 사용하려면 `javax.inject:javax.inject:1` 라이브러리를 추가해줘야함

  ```java
  @Autowired
  private Provider<PrototypeBean> provider;
  
  public int logic() {
        PrototypeBean prototypeBean = provider.get();
        prototypeBean.addCount();
        int count = prototypeBean.getCount();
        return count;
  }
  ```

ObjectProvider와 JSR-330 Provider 중에선 만약 코드를 스프링이 아닌 다른 컨테이너에서도 사용할 수 있어야한다면 JSR-330 Provider를 사용하면 된다. 하지만 그럴일은 거의 없다.

<br/>

## 웹 스코프

웹 스코프는 다음과 같은 특징이 있다.

- 웹 환경에서만 동작한다.
- 프로토타입과 다르게 스프링이 해당 스코프의 종료시점까지 관리한다. 따라서 종료 메서드가 호출된다.

그리고 종류는 다음과 같다.

- request : HTTP 요청하나가 들어오고 나갈 때까지 유지되는 스코프. 각각의 HTTP 요청마다 별도의 빈 인스턴스가 생성되고 관리된다.
- session : HTTP session과 동일한 생명주기를 가지는 스코프. 웹 브라우저별로 변수를 관리하고자 할 경우 사용한다.
- application : 서블릿 컨텍스트와 동일한 생명주기를 가지는 스코프
- websocket : 웹 소켓과 동일한 생명주기를 가지는 스코프

**Request 스코프 예시**

Request 스코프의 경우 매 요청마다 하나의 빈이 관리된다. 따라서 다음과 같이 로그를 남기는 빈에서, 요청을 구분하기 위해 사용될 수 있다.

```java
@Component
@Scope(value = "request")
public class MyLogger {
	private String uuid;
	private String requestURL;
  
	public void setRequestURL(String requestURL) {
		this.requestURL = requestURL;
	}
	public void log(String message) {
		System.out.println("[" + uuid + "]" + "[" + requestURL + "] " + message); 
	}
  
 	@PostConstruct
	public void init() {
		uuid = UUID.randomUUID().toString();
		System.out.println("[" + uuid + "] request scope bean create:" + this);
	}
  
	@PreDestroy
	public void close() {
		System.out.println("[" + uuid + "] request scope bean close:" + this);
	}
}
```

- 이 빈이 생성되는 시점에 자동으로 @PostConstruct 초기화 메서드를 사용해서 uuid를 생성해서 저장해둔다. 이 빈은 HTTP 요청 당 하나씩 생성되므로, uuid를 저장해두면 다른 HTTP 요청과 구분할 수 있다.
- 빈이 소멸되는 시점에 @PreDestroy 를 사용해서 종료 메시지를 남긴다. 따라서 요청이 끝난 시점을 기록할 수 있다.

**주의점**

만약 위와 같은 웹 스코프 빈을 주입받는 빈을 생성하고 애플리케이션을 실행시키면 다음과 같은 오류가 발생한다.

```
Error creating bean with name 'myLogger': Scope 'request' is not active for the
  current thread; consider defining a scoped proxy for this bean if you intend to
  refer to it from a singleton;
```

request 스코프 빈은 요청이와야 생성되는 빈이므로, 아직 생성도 안됐는데 다른 빈에서 주입받을 수 없다는 뜻이다.

이 문제를 해결해기 위해선 다음과 같은 방법이 있다.

- 웹 스코프 빈을 사용할 때마다 스프링빈에서 직접 조회 or Provider 사용

  - `ObjectProvider`로 필요할 때 꺼내어 사용하면 된다.
  - 웹 환경이 아니어서 빈이 존재하지 않는 경우가 있을 수 있는데, 그럴 땐 `getIfAvailable()` 메서드를 사용하면 된다. 빈을 사용할 수 없는 상황에선 null을 반환하는 메서드이다. 

- 프록시 사용

  ```java
  @Component
  @Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
  public class MyLogger
  ```

  - `@Scope` 애노테이션에 속성으로 proxyMode를 주어 프록시를 사용할 수 있다.
    - 적용대상이 클래스면 `TARGET_CLASS`, 인터페이스면 `INTERFACECS` 선택
  - 가짜 프록시 인스턴스를 만들고, HTTP Request와 상관없이 가짜 프록시 인스턴스를 다른 빈에 미리 주입하는 방식이다.

**프록시 동작원리**

- 스프링 컨테이너는 CGLIB라는 바이트코드를 조작하는 라이브러리를 사용해서, MyLogger를 상속받은 가짜 프록시 객체를 생성한다.
- 그리고 스프링 컨테이너에 "myLogger"라는 이름으로 진짜 대신에 이 가짜 프록시 객체를 등록한다.
- **가짜 프록시 객체는 요청이 오면 그때 내부에서 진짜 빈을 요청하는 위임 로직**이 들어있다.

![image-20221113224321240](https://images.velog.io/images/yh_lee/post/473753dc-2164-4978-a7ef-61e17ce5bf77/Screen%20Shot%202022-02-10%20at%202.00.36%20PM.png)

- 가짜 프록시 객체는 내부에 진짜 myLogger를 찾는 방법을 알고 있다. 따라서 클라이언트가 `myLogger.logic()`을 호출하면 사실은 가짜 프록시 객체의 메서드를 호출한 것이다.
- 가짜 프록시 객체는 원본 클래스를 상속받아서 만들어졌기 때문에 이 객체를 사용하는 클라이언트 입장에서는 사실 원본인지 아닌지도 모르게, 동일하게 사용할 수 있다(다형성)



<br/>

## 출처

- [김영한의 스프링 핵심원리 기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8)
- https://beoks.tistory.com/6
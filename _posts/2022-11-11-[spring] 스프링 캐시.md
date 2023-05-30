---
title: 스프링 캐시
author: 신성일
date: 2022-11-11 18:19:26 +0900
categories: [study, spring]
tags: [spring]
---

# **[spring] 스프링 캐시**

캐시란 반복적으로 데이터를 불러올 때 지속적으로 DBMS 혹은 서버에 요청하는 것이 아닌 메모리에 데이터를 저장하였다가 데이터를 불러다가 쓰는 것을 말한다. 서버나 DBMS의 부담을 줄여주고, 메모리에 저장되어있기 때문에 많은 시스템에서 사용할수 있다.

캐시는 `Long Tail 법칙`에 따라 시스템 리소스 사용의 대부분을 차지하는 20%의 요청을 캐싱하면 시스템 전체가 매우 빨라진다고 할 수 있다.

![17110B4350CC5EC51D](https://jaehun2841.github.io/2018/11/07/2018-10-03-spring-ehcache/17110B4350CC5EC51D.jpeg)

<Br/>

## **CacheManager**

스프링 부트에서 spring-boot-starter-cache을 추가하여 구성할 수 있는 캐시 매니저이다. 기본적으로 별도의 추가적인 서드파티 모듈이 없는 경우에는 Local Memory에 저장이 가능한 ConcurrentMap 기반인 `ConcurrentMapCacheManager`가 빈으로 자동 생성된다.

이 외에도 EHCache, Redis 등의 서드 파티 모듈을 추가하면 `EHCacheCacheManager`, `RedisCacheManager`를 빈으로 등록하여 사용할 수 있다. 이렇게 하면 별도로 다른 설정 없이도 단순 Memory cache가 아닌 cache server를 대상으로 캐시를 저장할 수 있도록 지원하고 있다.

스프링 3.2 이상에서는 7개의 CacheManager 구현체를 지원한다

-  SimpleCacheManager
-  NoOpCacheManager
-  ConcurrentMapCacheManager
-  CompositeCacheManager : 한 개 이상의 캐시매니저를 선택해야할 떄 사용
-  EhCacheCacheManager
-  RedisCacheManager (스프링 Redis)
-  GemfireCacheManager (스프링 GemFire)

### SimpleCacheManager 사용처

SimpleCacheManager는 단순히 캐시를 등록하여 사용하는데 사용할 수 있지만, 다른 캐시매니저와는 달리 다양한 캐시를 조합하여 등록할 수 있다는 점에서 편리할 수 있다.

```java
// ConcurrentMapCacheManager
// 캐시의 이름만 등록할 수 있고, 모두 ConcurrnetMapCache로 등록된다.
new ConcurrentMapCacheManager("comment", "category", "user");

// SimpleCacheManager
// 캐시를 직접 등록하여, 다양한 조합의 캐시로 사용할 수 있다.
SimpleCacheManager cm = new SimpleCacheManager();
cm.setCaches(Arrays.asList(
      defaultCache(),
      exceptionCache()));
```

<br/>

## **스프링 캐시 애노테이션**

### **@EnableCache**

Annotation 기반 캐싱 설정을 사용함을 명시하는 애노테이션이다. (내부적으로는 Sprign AOP 사용)

**속성**

-  proxyTargetClass : 클래스 기반의 Proxy 생성 여부. 디폴트는 false
   -  false : JDK Dynamic Proxy 사용. (interface 기반)
   -  true : CGLIB Proxy 사용 (class 기반)
-  Mode : 위빙 모드에 대한 설정. 디폴트는 PROXY
   -  PROXY : 기본의 spring AOP 방식을 이용한 RTW 방식 사용
   -  ASPECTJ : aspectj 라이브러리를 이용한 CTW, LTW 방식 지원
-  order : AOP order 설정. 디폴트는 Integer.MAX_VALUE

<br/>

### **@Cacheable**

```java
@Target(value={TYPE,METHOD})
@Retention(value=RUNTIME)
@Inherited
@Documented
public @interface Cacheable
```

메서드에 붙어, 해당 메서드가 호출된 결과가 캐시 될 수 있다고 알려주는 애노테이션이다.

**동작**

지시된 메서드가 호출될 때마다, 주어진 파라미터로 이미 호출된 적이 있는지 확인하는 과정이 추가된다. 기본적으론 메서드의 파라미터를 캐시 키로 사용하지만, `SpEL expression`이 `key()`로 적용될 수 있다. 또는 커스텀 [KeyGenerator](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/cache/interceptor/KeyGenerator.html) 구현이 적용될 수도 있다.

만약 주어진 캐시 키로 값을 찾을 수 없다면, 메서드는 정상적으로 호출되고 반환값을 만든다. 그리고 그 반환값은 캐시 저장소에 저장된다.

**주의** : `Optional`로 감싸진 값은 자동으로 unwrap 된다. Optional 값이 존재하면 그 값이 저장되고, 없으면 null이 저장된다.

**속성**

-  cacheManager : 사용할 CacheManager를 지정
-  cacheNames : 메서드의 반환값이 저장될 떄 사용될 캐시의 이름들
-  cacheResolver : 커스텀 [CacheResolver](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/cache/interceptor/CacheResolver.html)의 빈 이름(CacheResolver : Cache키에 대한 결과값을 돌려주는 리졸버)
-  condition : 메서드 결과값이 캐싱되도록 하는 조건. SpEL로 표현
-  unless : 메서드 결과값이 캐싱되지 않도록 하는 조건. SpEL로 표현
   -  ex) `unless = “#id == null”` --> id가 null일 땐 캐싱이 일어나지 않음.
-  key : 동적으로 키를 계산할 때 사용되는 SpEL 표현. 같은 캐시명을 사용할 때 구분되는 구분 값이다.
-  keyGenerator : 커스텀 [KeyGenerator](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/cache/interceptor/KeyGenerator.html)의 빈 이름. 특정 로직에 의해 cache key를 만들고자할 때 사용한다. key와 함께 사용할 수 없음.
-  sync : 복수의 스레드가 같은 키로 값을 가져오려고 할 때 동기화할 것인지 표시
-  value : [cacheNames()](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/cache/annotation/Cacheable.html#cacheNames--)의 alias

### **@CacheEvict**

메서드 실행 시, 해당 캐시를 삭제

**속성**

-  Value, cacheName : 삭제할 캐시 명
-  key : 같은 캐시명을 사용할 때 구분되는 구분값.
-  keyGenerator
-  cacheManager
-  cacheResolver
-  condition : SpEL 표현식을 통해 특정 조건에 부합하는 경우에만 캐시 사용. 연산 조건이 true일 때문 캐시 삭제
-  allEntries : Cache key에 대한 전체 데이터 삭제 여부
-  beforeInvocation : true면 메서드 실행 이전에 캐시 삭제. false면 메서드 실행 이후 삭제

### **@CachePut**

-  캐시가 존재하는지 검사하지않고 항상 메서드 실행하고 결과값을 캐시에 저장. 캐시를 갱신할 때 사용한다.
-  보통 @Cacheable과 @CachePut 을 동시에 사용하지 않는다. (실행 순서에 따라 다른 결과가 나올 수 있기에)
-  @CachePut은 캐시 생성용으로만 사용한다.

**속성**

-  Value, cacheName : 삭제할 캐시 명
-  key : 같은 캐시명을 사용할 때 구분되는 구분값.
-  keyGenerator
-  cacheManager
-  cacheResolver
-  condition : SpEL 표현식을 통해 특정 조건에 부합하는 경우에만 캐시 사용. 연산 조건이 true일 때문 캐시 삭제
-  unless : 메서드 결과값이 캐싱되지 않도록 하는 조건. SpEL로 표현

### **@Caching**

-  @Cacheable, @CacheEvict, @CachePut을 여러개 지정해야하는 경우 사용.
-  조건식이나 표현식이 다른 경우에 사용됨.
-  여러가지의 key에 대한 캐시를 중첩적으로 삭제해야할 때 사용.

**속성**

-  cacheable[] : 등록할 @Cacheable 등록
-  put[] : 등록할 @CacheEvict 등록
-  evict[] : 등록할 @CachePut 등록

### **@CacheConfig**

-  클래스 단위로 캐시 설정을 동일하게 하는데 사용
-  보통 CacheManager가 여러 개인 경우에 사용

**속성**

-  cacheNames : 캐시명
-  keyGenerator
-  cacheManater
-  cacheResolver

<br/>

## **사용**

`org.springframework.cache.annotation` 아래 있는 애노테이션으로, `spring-boot-starter-cache` 의존성을 추가해줘야한다.

```build.gradle
compile('org.springframework.boot:spring-boot-starter-cache')
```

### **config 추가**

```java
@Configuration
@EnableCaching
public class CachingConfig {
    @Bean
    public CacheManager cacheManager() {
        SimpleCacheManager cacheManager = new SimpleCacheManager();
        cacheManager.setCaches(Arrays.asList(
                new ConcurrentMapCache("article")));
        return cacheManager;
    }
}
```

### **캐시 적용**

```java
@Service
public class ArticleService {
    @Cacheable(value = "article")
    public String getArticles(long articleId){
			String article = "";

			System.out.println( code +  " article을 조회합니다 ...");
			if(articleId == 1){
				article = "첫번째 아티클"
			}
			else if(articleId == 2){
				article = "두번째 아티클";
			}
			else if(articleId == 3){
				article = "세번째 아티클";
			}
    }
}
```

<br/>

## **출처**

https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/cache/annotation/Cacheable.html

https://bob-full.tistory.com/23

https://jaehun2841.github.io/2018/11/07/2018-10-03-spring-ehcache/#Cache%EB%9E%80

https://shinsunyoung.tistory.com/50

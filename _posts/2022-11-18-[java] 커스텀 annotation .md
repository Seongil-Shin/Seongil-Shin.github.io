---
title: 커스텀 Annotation
author: 신성일
date: 2022-11-18 18:19:26 +0900
categories: [study, java]
tags: [java]
---

## Annotation

애노테이션은 Java 5 부터 등장한 기능으로, 사전적의미는 "주석"이지만, 클래스나 메서드 등 타켓에 라벨을 붙여준다. 비즈니스 로직에는 영향을 주지는 않지만, 해당 타켓의 연결 방법이나 소스 코드의 구조를 변경할 수도 있다. 애노테이션은 소스코드에 메타데이터를 삽입하는 것이기 때문에 잘 이용하면 구독성뿐만 아니라 체계적인 소스코드 구성에도 도움을 준다.

```java
// 애노테이션 예시
@Service
public class ArticleService
```

<br/>

## 커스텀 Annotation

커스텀 애노테이션은 메타 애노테이션을 사용하여 다음과 같은 구조를 가진다.

```java
@Target({ElementType.[적용대상]})
@Retention(RetentionPolicy.[정보유지되는 대상])
public @interface [어노테이션명] {
  public 타입 elementName() [default 값]
  ...
}
```

- `@Target`과 `@Retention` 등과 같은 메타 애노테이션으로 애노테이션의 성질을 정의한다.

- 애노테이션이 가질 수 있는 필드 타입은 `enum`, `String`, 원시형, 원시형의 배열만 사용할 수 있다.

- 다음처럼 다른 애노테이션을 추가할 수도 있다.

  ```java
  @Target({ElementType.[적용대상]})
  @Retention(RetentionPolicy.[정보유지되는 대상])
  @Service
  @RequiredArgsConstructor
  public @interface [어노테이션명]
  ```

### 메타 애노테이션

- @Retention : 애노테이션의 라이프 사이클을 정의하는 애노테이션으로, 애노테이션이 언제까지 메모리 상에 살아있을 건지 정의
  - RetentionPolicy.SOURCE : 컴파일 전까지만 유효
  - RetentionPolicy.CLASS : 컴파일러가 클래스를 참조할 때까지 유효
  - RetentionPolicy.RUNTIME : 컴파일 이후 런타임 때도 JVM에 의해 참조 가능함
- @Target : 애노테이션을 적용할 위치 선택
  - ElementType.PACKAGE : 패키지 선언
  - ElementType.TYPE : 타입 선언
  - ElementType.ANNOTATION_TYPE : 어노테이션 타입 선언
  - ElementType.CONSTRUCTOR : 생성자 선언
  - ElementType.FIELD : 멤버 변수 선언
  - ElementType.LOCAL_VARIABLE : 지역 변수 선언
  - ElementType.METHOD : 메서드 선언
  - ElementType.PARAMETER : 전달인자 선언
  - ElementType.TYPE_PARAMETER : 전달인자 타입 선언
  - ElementType.TYPE_USE : 타입 선언
- @Documented : 해당 애노테이션이 붙은 대상을 javadoc에 포함시킴
- @Inherited : 애노테이션의 상속을 가능하게 함
- @Repeatable : Java8부터 지원하며, 연속적으로 애노테이션을 선언할 수 있게함

### 속성 정의

애노테이션을 정의할 땐 마치 인터페이스처럼 다양한 속성을 정의할 수 있다.

```java
public @interface MyAnnotation {
  public String name() default "";
  public int age() default 20;
  public String value() default "";
    ...
}
```

이 중에서 `value`라는 이름을 가진 속성은 특별한 속성으로, 애노테이션을 사용할 때 아무런 속성이름 없이 하나의 값만 사용하면 자동으로 `value` 속성에 할당된다.

```java
// 여러 속성을 부여할 때
@MyAnnotation(name="Shin", age=20, value="this will be assigned to value")
// 한가지 값만 넣으면 value에 할당된다.
@MyAnnotation("this will be assigned to value")
```

<br/>

## 자바 리플랙션

어플리케이션 실행시 커스텀 애노테이션을 사용한 곳과 지정한 값들을 얻어오려면 **자바 리플랙션**을 사용해야한다.

```java
Method method = MyClass.class.getMethod("doThis"); //자바 리플렉션 getMethod로 메서드 doThis를 얻어온다
Annotation[] annotations = method.getDeclaredAnnotations(); //메서드에 선언된 어노테이션 객체를 얻어온다

//메서드 doThat에 선언된 MyAnnotation의 어노테이션 객체를 얻어온다
Annotation annotation = MyClass.class.getMethod("doThat").getAnnotation(MyAnnotation.class);
```

<br/>

## 출처

https://www.nextree.co.kr/p5864/

https://jeong-pro.tistory.com/234

https://velog.io/@potato_song/Java-%EC%96%B4%EB%85%B8%ED%85%8C%EC%9D%B4%EC%85%98-%EC%BB%A4%EC%8A%A4%ED%85%80-%EC%96%B4%EB%85%B8%ED%85%8C%EC%9D%B4%EC%85%98-%EB%A7%8C%EB%93%A4%EA%B8%B0

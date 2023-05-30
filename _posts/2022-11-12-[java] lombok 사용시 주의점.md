---
title: lombok 사용시 주의점
author: 신성일
date: 2022-11-12 19:19:26 +0900
categories: [study, java]
tags: []
---



lombok은 `@Getter`, `@Setter` 같은 애노테이션 기반으로 Getter/Setter 메서드를 자동으로 생성해주는 편리한 라이브러리이다.

하지만 편리함에 남용하는 애노테이션들이 있다.

## **lombok 사용 시 주의해야할 애노테이션**

**@AllArgsConstructor**

클래스에 존재하는 모든 필드에 대한 생성자를 자동으로 생성한다. 

```java
@AllArgsConstructor
public class User {
  private String name;
  private String userId;
  
  // 자동 생성
  public User(String name, String userId) {
    this.name = name;
    this.userId = userId;
  }
}
```

이때 **생성자의 파라미터 순서는 필드가 선언된 순서와 같다**. 여기서 문제가 발생할 수 있다. 만약 위 예시에서 개발자가 name과 userId의 순서를 변경한다면, lombok은 생성자의 파라미터 순서도 바꾼다. 이때 만약 다음과 같은 메서드가 애플리케이션 어딘가에서 사용중이었다면 어떠한 오류도 발견되지 않고 넘어갈 것이다.

```java
User tempUser = new User("name", "userId");
```

<br/>

**@RequiredArgsConstructor**

클래스의 final 필드에 대한 생성자를 생성한다. `@AllArgsConstructor`와 같은 문제를 가지고 있다. 

하지만 빈 주입으로 사용할 경우엔 스프링에서 객체를 생성하므로 크게 신경 안써도 된다.

<br/>

**@EqualsAndHashCode**

equals 메서드와 hashcode 메서드를 생성한다. equals 메서드로 두 객체의 필드들이 서로 같은 값을 가지는지 검사하고, hashcode 메서드로 실제로 같은 객체인지 검사한다.

편리하지만, `Mutable 객체`에 아무런 파라미터 없이 그냥 사용하면 문제가 된다. 객체를 Set에 저장한 뒤 필드 값을 변경하면 hashCode가 변경되면서 이전에 저장한 객체를 찾을 수 없는 문제가 발생하기 때문이다.  

```java
@EqualsAndHashCode
public class User {
  private String name;
  private String userId;
  
  public void setName(String name) {
    this.name = name;
  }
}
  
User userInfo = new User("Park", "qwer1234");
  
Set<User> userSet = new HashSet<>();
userSet.add(userInfo); // Set에 객체 추가
  
System.out.println("변경전 : " + userSet.contains(userInfo)); // true
  
userInfo.setName("Shin"); // price 값 변경
System.out.println("변경후 : " + userSet.contains(userInfo)); // false    
```

Set에서는 동일한 객체임을 검사할 때 hashCode() 메서드를 사용하는데, equals()로 같은 객체는 hashCode()로도 같아야하므로 필드 값이 변경될 경우 hachCode도 달라지기 때문이다.

따라서 mutable 객체에서는 다음과 같이 동등성 비교에 필요한 필드를 명시하는 형태로 사용한다.

```java
@EqualsAndHashCode(of={"userId"})
public class User {
  private String name;
  private String userId;
  
  public void setName(String name) {
    this.name = name;
  }
}
```

하지만 막상 개발을 하다보면 immutable 필드를 대상으로만 equals, hashCode를 만들기는 어렵다. 최소한 꼭 필요하고 일반적으로 변하지 않는 필드에 대해서만 만들도록 노력해야한다.

<Br/>

**@Value**

immutable 클래스를 만들어주는 조합 애노테이션으로, @EqualsAndHashCode, @AllArgsConstructor가 포함된다. 

@EqualsAndHashCode는 immutable 클래스이므로 문제가 되지 않지만, @AllArgsConstructor는 위에서 언급한 문제가 있다.

<br/>

**@Data**

@Getter, @Setter, @RequiredArgsConstructor, @EqualsAndHashCode, @ToString를 포함하는 조합 애노테이션이다.

위에서 언급한 문제들을 가지고 있다.

<br/>

## **출처**

https://velog.io/@rosa/Lombok-%EC%A7%80%EC%96%91%ED%95%B4%EC%95%BC-%ED%95%A0-annotation

https://kwonnam.pe.kr/wiki/java/lombok/pitfallhttps://bkjeon1614.tistory.com/657

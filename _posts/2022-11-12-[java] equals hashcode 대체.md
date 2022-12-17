---
title: equals(), hashCode()
author: 신성일
date: 2022-11-12 15:19:26 +0900
categories: [study, java]
tags: [java, equals]
---



equals()와 hashCode() 메서드는 모든 자바 객체의 부모인 Object 클래스에 정의되어있다. 따라서 모든 자바 객체는 equals()와 hashCode() 메서드를 가지고 있다.

## **equals()**

현재 객체와 파라미터로 들어온 객체가 같은 지 검사하기 위해 사용한다. 기본적으로는 두 객체의 메모리 주소가 같아야 동일한 객체가 된다.

```java
public boolean equals(Objecet obj) {
  return (this == obj);
}
```

이러한 equals 객체를 오버라이드하여 필드값이 같은 객체를 같은 객체라고 판단하게 할 수 있다.

```java
@AllArgsConstructor
@Getter
public class User {
  private String name;
  
  @Override
  public boolean equals(Object obj) {
    if(obj instanceof User) {
      return this.getName().equals(obj.getName()); 
    } else {
      return false;
    }
  }
}
```

## **hashCode()**

런타임 중에 객체의 유일한 integer 값을 반환한다. Object 클래스에서는 heap에 저장된 객체의 메모리 주소를 반환하도록 되어있다.

```java
public native int hashCode();
```

native 키워드

- 메소드가 JNI(Java Native Interface)라는 native code를 이용해 구현되었음을 의미
- native는 메소드에만 적용가능한 제어자로, C or C++ 등 Java가 아닌 언어로 구현된 부분을 JNI를 통해 Java에서 이용하고자 할 때 사용
- 일반 개발자는 사용할 수 없다. 
- hashCode에서는 HashTable과 같은 자료구조를 사용할 때 데이터가 저장되는 위치를 결정하기 위해 사용한다.

<br/>

## **equals() 와 hashCode()의 관계**

동일한 객체는 동일한 메모리 주소를 갖는다는 것을 의미하므로, 동일한 객체는 동일한 해시태그를 가져야한다. 그렇기 때문에 만약 우리가 equals() 메서드를 오버라이드한다면, hashCode() 메서드도 오버라이드 되어야한다. equals()와 hashCode()는 아래와 같은 관계를 만족해야한다.

- Java 프로그램을 실행하는 동안 equals()에 사용된 정보가 수정되지 않았다면, hashCode()는 항상 동일한 정수값을 반환해야한다.
- **두 객체가 equals()에 의해 동일하다면, 두 객체의 hashCode() 값도 일치해야한다.**
- 두 객체가 equals()에 의해 동일하지 않다면, 두 객체의 hashCode() 값도 일치하지 않아도 된다.

<br/>

## **출처**

https://mangkyu.tistory.com/101

https://kwonnam.pe.kr/wiki/java/equals_hashcode
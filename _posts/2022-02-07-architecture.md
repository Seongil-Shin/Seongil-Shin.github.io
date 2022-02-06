---
title: singleton
author: 신성일
date: 2022-02-07 01:00:00 +0900
categories: [cs, design pattern]
tags: [cs, 디자인패턴]
---

## **Singleton 패턴**

애플리케이션에서 인스턴스를 하나만 만들어 사용하기 위한 패턴이다.

커넥션 풀, 스레드 풀, 디바이스 설정 객체 등의 경우, 인스턴스를 여러 개 만들게하면 자원을 낭비하게 되거나 버그를 발생시킬 수 있으므로 오직 하나만 생성하고 그 인스턴스를 사용하도록 하는 것이 목적이다.

<br/>

### **구현**

하나의 인스턴스만을 유지하기 위해 인스턴스 생성에 특별한 제약을 걸어둬야한다.

1. new를 실행할 수 없도록 생성자에 private 접근 제어자를 지정
2. 유일한 단일 객체를 반환할 수 있도록 정적 메소드를 지원.
3. 유일한 단일 객체를 참조할 정적 참조변수 필요

```java
public class Singleton {
    private static Singleton singletonObject;

    private Singleton() {}

    public static Singleton getInstance() {
        if (singletonObject == null) {
            singletonObject = new Singleton();
        }
        return singletonObject;
    }
}
```

위 코드는 멀티 스레딩환경에서 싱글턴 패턴을 적용할떄 문제가 발생한다. 동시에 접근하다가 하나만 생성되어야하는 인스턴스가 두개 생성될 수 있기때문이다. 따라서 `getInstance()` 메소드를 동기화 시켜야 한다.

```java
public class Singleton {
    private static Singleton singletonObject;

    private Singleton() {}

    public static synchronized Singleton getInstance() {
        if (singletonObject == null) {
            singletonObject = new Singleton();
        }
        return singletonObject;
    }
}
```

좀 더 효율적으로 하려면, `DCL(double checking locking)`을 써서 `getInstance()` 에서 동기화 되는 영역을 줄일 수 있다.

```java
public class Singleton {
    private static volatile Singleton singletonObject;

    private Singleton() {}

    public static Singleton getInstance() {
        if (singletonObject == null) {
            synchronized (Singleton.class) {
                if(singletonObject == null) {
                    singletonObject = new Singleton();
                }
            }
        }
        return singletonObject;
    }
}
```

이때 위 코드는, 멀티코어 환경에서 하나의 CPU를 제외하고는 다른 CPU가 lock이 걸리게 된다. 따라서 다른 방법이 필요하다.

```java
public class Singleton {
    private static volatile Singleton singletonObject = new Singleton();

    private Singleton() {}

    public static Singleton getSingletonObject() {
        return singletonObject;
    }
}
```

`volatile` : 컴파일러가 특정변수에 대해 옵티마이져가 캐싱을 적용하지 못하도록 하는 키워드이다.

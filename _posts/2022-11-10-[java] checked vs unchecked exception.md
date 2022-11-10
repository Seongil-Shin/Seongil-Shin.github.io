---
title: checked vs unchecked exception
author: 신성일
date: 2022-11-10 18:24:26 +0900
categories: [study, java]
tags: [java, exception]
---

# **[java] checked vs unchecked exception**

자바에서 프로그램에 이상이 있을 때 던져지는 Throwable은 Error와 Exception이 있다. 

![throwable](https://github.com/cheese10yun/TIL/blob/master/assets/excpetion-class-diagram.png?raw=true)

- Error : 시스템이 비정상적인 상황에 있다는 것을 의미. 시스템 레벨에서 발생하는 심각한 오류이기 때문에 개발자가 미리 예측하거나 처리할 수 없다.
  - ex) OutOfMemoryError, ThreadDeath
- Exception : 개발자들이 만든 애플리케이션 코드에서 예외가 발생했다는 것을 의미한다.
  - Checked Exception : 반드시 예외처리해야하는 예외. 스프링 트랜잭션 Rollback이 진행되지 않음
    - Ex) IOException, SQLException
  - Unchecked Exception : 예외 처리해주지 않아도 되는 예외. 스프링 트랜잭션 Rollback 진행
    - ex) NullPointerException, IllegalArgumentException

그림에서 보이듯이 Unchecked Exception은 RuntimeException을 상속받는다. 하지만 Checked Exception은 상속받지 않는다. 이것이 두 Exception을 구분하는 중요한 포인트이다.

<br/>

## **checked vs unchecked Exception**

**반드시 처리해줘야하는가**

- Checked exception : 반드시 명시적으로 처리해야하는 예외이다. 따라서 try-catch로 처리하든, throws로 호출한 메서드로 예외를 날리지 않으면 컴파일 오류가 발생한다.
- unchecked exception : 명시적인 예외처리가 필요없다. 따라서 예외처리를 해주지 않아도 상관없다.

**스프링 트랜잭션 Rollback 여부**

- Checked exception : 예외가 날아가도 rollback이 진행되지 않는다. 예외를 감지할 수 있으니 개발자에게 복구를 맡기기 때문이다.(이는 스프링이 EJB 관습을 따르기 때문이라고 한다.)
- unchecked exception : 예외 발생 시 rollback이 진행된다.

```java
@Service
@RequiredArgsConstructor
@Transactional
public class MemberService {
  private final MemberRepository memberRepository;

  // (1) RuntimeException 예외 발생
  public Member createUncheckedException() {
    final Member member = memberRepository.save(new Member("yun"));
    if (true) {
      throw new RuntimeException();
    }
    return member;
  }

  // (2) IOException 예외 발생
  public Member createCheckedException() throws IOException {
    final Member member = memberRepository.save(new Member("wan"));
    if (true) {
      throw new IOException();
    }
    return member;
  }
}
```

<br/>

## **checked exception 처리 전략**

**스프링 트랜잭션 Rollback 관련**

checked exception이 날라가도 스프링은 트랜잭션 Rollback을 하지 않는다. 개발자에게 맡기기 때문이다.  실제로 Transaction rollback을 적용할 때 쓰이는 `@Transactional` 애노테이션의 rollbackFor 옵션의 기본값은 다음과 같다.

```java
@Transactional(rollbackFor = {RuntimeException.class, Error.class})
```

하지만 현실적으로 개발자가 처리해줄 수 없는 경우도 있다. 이럴 때는 다음과 같은 방법이 있다.

- try-catch로 잡은 후, RuntimeException으로 날리기
  - 이때는 왜 exception이 발생하였는지 명확하게 보내는 것이 좋다.
- `@Transactional(rollbackFor={Exception.class, Error.class})`로 변경하여 모든 예외를 대산으로 롤백하기.
- `RollbackRuleAttribute` 를 이용해 롤백 규칙 추가하기

2,3번째 방법을 사용하면 checked exception에서도 롤백을 진핼할 수 있으나, 스프링 기본 설정을 건드려야한다. 이렇게 기본 설정을 건드는 것은 향후 문제가 될 수 있다. 따라서 웬만하면 unchecked exception으로 바꿔 날리는 것이 좀 더 안전하며, 명확한 exception으로 바꿔 보낸다면 더 좋은 설계가 된다고 생각한다

**unchecked exception으로 바꿔 날리기**

Checked exception을 throws로 상위 메서드도 throws로 날려야하고, 그 상위 메서드도 throws로 날려야하는 문제가 발생한다. 이는 좋지 않은 패턴이다. 따라서 반드시 예외는 처리 후 더욱 구체적인 unchecked exception으로 날려야한다.

```java
// throws 지옥
public void root() throws IOException {
  aaa();
}

public void aaa() throws IOException {
  bbb();
}

public void bbb() throws IOException {
 	throw new IOException();
}

// 처리 후 
public void root() {
  aaa();
}

public void aaa() {
  bbb();
}

public void bbb() {
  try {
 		throw new IOException();
  } catch(IOException e) {
    throw new JsonDeserializeFailed(e.getMessage());
  }
}
```

<br/>

## **출처**

- https://cheese10yun.github.io/checked-exception/
- https://jangjjolkit.tistory.com/m/3
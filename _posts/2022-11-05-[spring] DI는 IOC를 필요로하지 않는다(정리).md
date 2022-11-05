---
title: [spring] Pure DI - IoC가 없는 DI
author: 신성일
date: 2022-11-05 18:34:26 +0900
categories: [study, spring]
tags: [spring, di, ioc]
---

이 포스트는 [참고문서](https://jwchung.github.io/DI%EB%8A%94-IoC%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EC%A7%80-%EC%95%8A%EC%95%84%EB%8F%84-%EB%90%9C%EB%8B%A4)를 필자가 이해하기 쉽게 정리한 내용입니다.

## DI는 IoC를 사용하지 않아도 된다

### DI(Dependency Injection)

다음 `PizzaStore` 클래스는 정해진 개수의 `Pizza`인스턴스를 가지고 있다. 만약 `Pizza`가 더 필요하면 `PizzaStore`를 직접 수정해야하다. 이는 OCP를 위반하며 DI로 해결해줄 수 있다

```java
public class PizzaStore
{
    private readonly Pizza[] pizzas =
        new Pizza[]
        {
            new Pizza(),
            new Pizza(),
            new Pizza()
        };

    public void Sell(int count)
    {
```

다음과 같이 인수를 취하는 형태의 DI로 해결 가능하다

```java
public class PizzaStore
{
    private readonly Pizza[] pizzas;

    public PizzaStore(Pizza[] pizzas)
    {
        this.pizzas = pizzas;
    }

    public void Sell(int count)
    {
```

이처럼 DI는 추상화를 해치지 않고 의존성을 인수로 넘겨주는 방법을 뜻한다.

> [Rúnar Bjarnason](https://www.youtube.com/watch?v=ZasXwtTRkio) : Dependency Injection is really just a pretentious way to say ‘taking an argument’.

### DIP(Dependency Inversion Principle)

> 고차원 모듈은 저차원 모듈에 의존해서는 안된다. 이 둘은 추상화된 것에 의존해야한다.
>
> 추상화된 것은 구체적인 것에 의존하면 안된다.

위 `PizzaStore` 클래스는 현재 `Pizza ` 구현체에 의존하고 있다. 이 형태는 DIP를 위반하고 있는 것으로, 다음과 같이 변경되어야한다.

![dip위반](../assets/img/[spring] DI는 IOC를 필요로하지 않는다(정리)/DIP-violation.png)

![dip 지킴](../assets/img/[spring] DI는 IOC를 필요로하지 않는다(정리)/DIP-resolved.png)

DI는 DIP를 사용한다고 생각하지만 이 둘은 별개다. DI를 사용하면서 DIP가 필요없는 경우도 많다. 아래 `CachedUserStore` 클래스에서는 일정기간(`duration`) 동안 캐쉬하는 기능을 제공한다. 이때 `duration`은 DI를 통해 주입되지만 DIP가 필요한 것은 아디ㅏ.

```java
public interface IUserStore
{
    User Find(string userId);
}

public class CachedUserStore : IUserStore
{
    ...

    public CachedUserStore(IUserStore innerStore, int duration)
    {
        ...
    }

    public User Find(string userId)
    {
```

<br/>

### Inversion of Control

IoC는 라이브러리와 프레임워크의 차이에서 알아볼 수 있다.

> 라이브러리 : 개발자가 원하는 때에 가져와서 사용할 수 있음. 개발자에게 제어권이 있다
>
> 프레임워크 : 개발자가 프레임워크에 맞추어 구성요소를 등록하면 프레임워크가 프로그램을 동작시킴. 제어권이 프레임워크에 있다.

다음과 같이 IoC 컨테이너가 의존성을 주입해준다면 이는 제어권이 IoC 컨테이너에 있는 것으로 IoC가 발생한 사레이다

```java
public static void Main()
{
    Type[] pizzaTypes = Assembly.GetEntryAssembly()
        .GetExportedTypes()
        .Where(typeof(IPizza).IsAssignableFrom);

    var builder = new IoCContainerBuilder();
    buildr.RegisterTypes(pizzaTypes);

    IoCContainer container = builder.Build();

    IPizza[] allKindsOfPizzas = container.Resolve<IPizza[]>();

    var pizzaStore = new PizzaStore(allKindsOfPizzas);

    ...
}
```

<br/>

### Pure DI

> Mark Seemann은 IoC 컨테이너를 사용하지 않는 DI에 대해 Pure DI라고 했다

DI를 사용한다고 무조건 IoC 컨테이너가 필요한 것은 아니다. 보통은 의존성 등록비용(개발자가 직접 의존성을 주입하는 것) 때문에 IoC 컨테이너로 의존성 주입하지만, IoC 컨테이너를 사용하는 방법에는 `Weakly typed` 문제가 있다.

Weakly typed 란, IoC 컨테이너에 의존성을 잘못 구성되였을 경우 컴파일 에러대신 런타임 에러가 발생하는 것이다.

따라서 Weakly typed 비용과 의존성 등록 비용 중 어떤 것이 더 큰지 생각하여 IoC 컨테이너로 DI를 구현할지 Pure DI를 사용할지 결정해야한다. (참고문서의 필자는 weakly typed 비용이 더 크다고 함)

중요한 것은 뭘 사용해야할지보다 IoC 컨테이너 없이 Pure DI를 사용할 수 있고, Pure DI에도 장점이 있다는 것이다.

[참고문서](https://jwchung.github.io/DI%EB%8A%94-IoC%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EC%A7%80-%EC%95%8A%EC%95%84%EB%8F%84-%EB%90%9C%EB%8B%A4)

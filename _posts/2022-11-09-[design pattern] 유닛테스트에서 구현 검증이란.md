---
title: 유닛테스트 구현 검증이란?
author: 신성일
date: 2022-11-09 18:19:26 +0900
categories: [study, design pattern]
tags: [design]
---

비즈니스 코드를 먼저 작성하고 유닛테스트를 작성할 때, 완성된 코드를 보고 테스트 코드를 작성하다보면 내부 구현에 맞추어 테스트를 작성하게 되고, 구현 자체를 검증하게 되는 일이 많이 발생하였다.

**유닛 테스트에서는 구현이 아닌 행동을 검증**(test behaviour, not implementation)해야한다. 따라서 구현을 검증한 테스트 코드를 고치려 했으나 어디까지가 구현을 검증한 것인지가 모호했다.

> A 메서드 안에서 B 메서드를 제대로 호출했는지를 보는 것도 구현 검증일까?
>
> 구현 검증을 피하고자 DB를 검사해도 되는 걸까?
>
> 최종 결과물을 내지 않는 void 메서드는 어떻게 테스트하지?

명확한 기준을 세우고 싶어 여러모로 조사하였다.

<br/>

## **구현을 검증하는 경우**

아래 내용은 [유닛테스트에서 내부구현 검증 피하기](https://jojoldu.tistory.com/614)라는 포스트에서 가져온 내용으로, 구현 검증이 무엇인지 이해하기 위해 살펴봤다.

### **상세 구현부를 다 검증하는 경우**

다음과 같은 클래스가 있다

```javascript
export class OrderAmountSum {
   minusSum: number = 0;
   plusSum: number = 0;

   constructor(amounts: number[]) {
      this.addPlusAmounts(amounts);
      this.addMinusAmounts(amounts);
   }

   get sumAmount(): number {
      return this.plusSum + this.minusSum;
   }

   addPlusAmounts(amounts: number[]): void {
      this.plusSum = amounts
         .filter((amount) => amount > 0)
         .reduce((before, current) => before + current);
   }

   addMinusAmounts(amounts: number[]): void {
      this.minusSum = amounts
         .filter((amount) => amount < 0)
         .reduce((before, current) => before + current);
   }
}
```

이 클래스에서 중요한 비즈니스 로직은 `sumAmount`에 있다. `addPlusAmounts`, `addMinusAmounts`은 부차적인 기능에 불과하다. 하지만 여기서 다음과 같이 부차적인 기능에 대한 테스트를 짜는 실수를 할 수 있다.

```javascript
it("plusSum에는 양수들의 총합이 등록된다", () => {
   const sut = new OrderAmountSum([1000, 300, -100, -500]);

   expect(sut.plusSum).toBe(1300);
});

it("minusSum에는 음수들의 총합이 등록된다", () => {
   const sut = new OrderAmountSum([1000, 300, -100, -500]);

   expect(sut.minusSum).toBe(-600);
});

it("전체 합계 금액을 구한다", () => {
   const sut = new OrderAmountSum([1000, 300, -100, -500]);

   expect(sut.sumAmount).toBe(700);
});
```

이렇게 되면 만약 다음과 같이 코드가 변경되었을 경우 `sumAmount` 테스트 말고는 전부 깨질 것이다.

```javascript
export class OrderAmountSum2 {
   amounts: number[];

   constructor(amounts: number[]) {
      this.amounts = amounts;
   }

   get sumAmount(): number {
      return this.amounts.reduce((before, current) => before + current);
   }
}
```

핵심 로직이 아닌 부차적인 기능까지 테스트하여 발생한 문제다. 따라서 `OrderAmountSum`의 테스트 코드는 다음과 같이 작성되어야한다.

```javascript
describe("OrderAmountSum2", () => {
   it("전체 합계 금액을 구한다", () => {
      const sut = new OrderAmountSum2([1000, 300, -100, -500]);

      expect(sut.sumAmount).toBe(700);
   });

   it("-금액들의 합계 금액을 구한다", () => {
      const sut = new OrderAmountSum2([-1000, -300, -100, -500]);

      expect(sut.sumAmount).toBe(-1900);
   });
});
```

<br/>

### **모의 객체 행위 검증하기**

다음과 같이 Order를 업데이트하거나 생성하는 메소드가 있다고 해보자.

```javascript
export class OrderService {
    constructor(private readonly orderRepository: OrderRepository) {
    }

    saveOrUpdate(order: Order): void {
        const savedOrder = this.orderRepository.findById(order.id);
        if (savedOrder) {
            this.orderRepository.update(order);
        } else {
            this.orderRepository.save(order);
        }
    }
}
```

이때 stub을 통한 모의 객체를 사용하여 다음과 같은 테스트를 작성할 수 있다.

```javascript
it("기존 주문이 있으면 새 정보로 갱신된다", () => {
   const savedOrder = Order.create(1000, LocalDateTime.now(), "");
   when(mockedRepository.findById(anyNumber())).thenReturn(savedOrder);
   const sut = new OrderService(instance(mockedRepository));

   sut.saveOrUpdate(createOrder(savedOrder, 200));

   verify(mockedRepository.update(anything())).called();
});
```

여기서의 문제는 `saveOrUpdate` 내부에서 **repository를 하나하나 어떻게 쓰는지 검증**했다는 점이다. 만약 다음과 같이 `saveOrUpdate`가 변경된다면 어떻게 될까?

```javascript
saveOrUpdate(order: Order): void {
    this.orderRepository.saveOrUpdate(order);
}
```

실제로는 잘 저장 또는 업데이트 되어도 테스트는 실패했다고 뜰 것이다. 따라서 테스트는 다음과 같이 진행되어야한다.

```javascript
it("[After] 기존 주문이 있으면 새 정보로 갱신된다", () => {
   const savedOrder = realRepository.save(
      Order.create(1000, LocalDateTime.now(), "")
   );
   const expectAmount = 200;

   const sut = new OrderService(realRepository);
   sut.saveOrUpdate2(createOrder(savedOrder, expectAmount));

   const result = realRepository.findById(savedOrder.id);

   expect(result.amount).toBe(expectAmount);
});
```

<br/>

## **의문 하나씩 살펴보기**

위에서 살펴본 경우를 보고 한번 글 초반에 작성했던 의문에 대한 답을 생각해보자

### A 메서드 안에서 B 메서드를 제대로 호출했는지를 보는 것도 구현 검증일까?

구현 검증이다. A 메서드는 B 메서드를 호출하든 안하든 A 메서드에 부여된 일을 수행하면 될 뿐이다.

만약 구현이 변경되어 A메서드는 B 메서드가 아닌 C 메서드를 호출하는 것으로 변경되었으나 기능은 기존과 동일할때, B 메서드에 의존한 테스트는 실패하게 된다. 이는 위 예시로 봤을 때 구현 검증으로 테스트가 깨진 사례에 속한다.

### **구현 검증을 피하고자 DB를 검사해도 되는 걸까?**

좋지 않은 패턴이다. DB에 의존하여 결과를 확인하는 것은 DB의 상태, 버전 등에 따라 테스트 결과가 달라질 수도 있다. 매번 결과가 다른 테스트가 작성되는 것이고, 이는 테스트의 `F.I.R.S.T` 원칙의 `Repeatable` 을 위반한 사례이다.

### **최종 결과물을 내지 않는 void 메서드는 어떻게 테스트하지?**

구현이 아닌 행동를 테스트해야하는 건 잘 알았다. 하지만 결과를 쉽게 알 수 없는 void 메서드일 경우는 어떻게 테스트 해야할까?

테스트 코드 작성 경험이 별로 없는 나에겐 빠르게 생각하기 어려웠다. 따라서 한번 관련 글들을 조사해보았다.

 <br/>

## **Test behaviour, not implementation**

`Test behaviour, not implementation`이라고 검색한 후 나온 포스트들을 하나씩 보며 구현을 검증하지 않고 행동을 검증하는 것이 무엇인지를 알아보았다

### **[Testing Behavior vs. Testing Implementation](https://launchscout.com/blog/testing-behavior-vs-testing-implementation)**

**Testing Behavior** : 정답이 왜 나오는 지는 모르겠는데, 이 환경에서 이 결과만 나오면 돼!

**Testing Implementation** : 결과는 신경안쓰고, 과정만 제대로 확인해!

실제로는 결과를 보며 테스트를 하지만, 너무 많은 mock와 stub에 의존하며 결과가 어떻게 나오는지에 대한 많은 assertions을 만들게 된다. 이러한 mocks와 stub은 시스템이 발전함에 따라 문제가 된다.

mocks와 stub가 나쁜 것은 아니다. mocks와 stub은 서로 다른 시나리오와 환경에서 테스트를 할 수 있게 해준다. 이 정도는 `supporting implementation`이며 `testing implementation`이 아니다. 하지만 너무 많은 assertions이 mocks와 stub에 의존하여 나온다면 문제가 된다.

**Testing behavior와 testing implementation이 같아지는 때**가 있다. 테스트하는 메서드가 coordinator 메서드일 경우 그렇다. 이러한 메서드는 결정을 내리고 다른 메서드나 클래스를 호출하는 역할을 한다. 이러한 메서드를 테스트할 때는 메서드가 적절히 불러졌는지 확인하는 것밖에 없다.

Testing begavior와 testing implementation이 같아지는 예시)

다음과 같이 terminalService에서 terminal list을 가져와 하나씩 `ContactTerminal` 메서드에 넣어 실행시키는 `FindTerminals` 메서드가 있다고 해보자.

```javascript
void FindTerminals() {
    terminals = terminalService.FindAll();
    foreach (var terminal in terminals)
        ContactTerminal(terminal);
}
```

`FindTerminals` 에 대한 테스트는 다음과 같다.` terminalService.FIndAll()` 메서드의 응답으로 3개의 terminal 을 응답하도록 stub 처리하고, FindTerminals을 실행시킨다. 그리고 정확히 3번 ContactTerminal이 실행되는지 확인하는 코드이다.

```javascript
var terminals = new List<Terminal>{ new Terminal(), new Terminal(), new Terminal() };
terminalServiceMock
    .Setup(f => f.FindAll())
    .Returns(terminals);

var activeTerminalLocatorMock = new Mock<ActiveTerminalLocator>(
        terminalServiceMock.Object,
        networkServiceFactoryMock.Object,
        backgroundWorkerFactory
);
activeTerminalLocatorMock.Object.FindTerminals(null);

activeTerminalLocatorMock.Verify(
    f => f.ContactTerminal(It.IsAny<Terminal>()),
    Times.Exactly(3)
);
```

위와 같은 테스트는 구현을 검증한 예시이다. 어떤 메서드가 몇번 호출되었는지 확인하는 코드이기 때문이다. 하지만 결과를 검증하는 예시이기도 하다. ContactTerminal이 주어진 terminal만큼 실행시키는 것이 FindTerminals의 목적이고, 그것을 검증하는 코드이기 때문이다. 예를 들어 FindTerminals가 다음과 같이 변경되어도, FindTerminals의 목적은 똑같기에 때문에 테스트 코드는 여전히 유효하다.

```javascript
void FindTerminals() {
    terminals = terminalService.FindAll();
    foreach (var terminal in terminals) {
        using(var worker = workerFactory.CreateWorker(){
            worker.DoWork += (o, ea) => ContactTerminal(terminal);	// 3번 실행됨.
            worker.RunWorkerAsync();
         }
     }
}
```

이와 같이 테스트 대상의 목적을 검증하는 것과 구현을 검증하는 것이 같아지는 때가 있다. 이때는 구현이 크게 변경되어도, 목적이 달라지지 않는다면 테스트 코드는 변화하지 않는다.

---

**포스트 후기**

이 포스팅에서 얻은 것은 테스트 대상의 목적을 확실히하고, 그 목적을 테스트한다면 구현을 테스트하더라도 위에서 걱정했던 테스트 코드를 변경하는 문제는 나타나지 않는다는 점이다.

예시에서 봤듯이 어떤 메서드를 몇번 호출하였는지를 확인하는 것은 구현 검증이지만, 목적 자체가 ContactTerminal을 호출하는 것이었기에 테스트 코드는 변화하지 않았다.

만약 ContactTerminal를 호출하지 않는 사양으로 메서드가 변경된다면 그것은 목적이 변한 것이니 애초에 메서드를 분리해서 구현하든가, 테스트 코드도 마땅히 바꿔야하는 일이다.

<br/>

### **[Michel - Test behavior, not implementation](https://craftsmen.nl/test-behaviour-not-implementation/)**

다음과 같이 간단한 메서드가 있다고 해보자.

```java
public List<Book> getAllBooks() {
	return bookRepository.getAllBooks();
}
```

이 메서드에 대한 테스트를 다음과 같이 작성할 수 있다. 이는 bookRepository의 메서드가 제대로 호출되었는지 확인하는 것이므로 구현을 테스트한 것에 속한다.

```java
private Book lord_of_the_rings = new Book("Lord of the rings");

@Test
public void reallyInterestedInWhatThisServiceIsDoing() {
	BookRepository bookRepository = mock(BookRepository.class);
   when(bookRepository.getAllBooks()).thenReturn(Collections.singletonList(lord_of_the_rings));
	BookService bookService = new BookService(bookRepository);

	assertThat(bookService.getAllBooks(), containsInAnyOrder(lord_of_the_rings));
	verify(bookRepository, times(1)).getAllBooks();
}
```

우리는 behaviour를 테스트해야한다. 따라서 위 테스트는 아래와 같이 behaviour만 테스트하는 형태로 변경되어야한다.

```java
private Book lord_of_the_rings = new Book("Lord of the rings");

@Test
public void shouldGetAllBooks() {
	BookRepository bookRepository = mock(BookRepository.class);
   when(bookRepository.getAllBooks()).thenReturn(Collections.singletonList(lord_of_the_rings));
	BookService bookService = new BookService(bookRepository);

	assertThat(bookService.getAllBooks(), containsInAnyOrder(lord_of_the_rings));
}
```

물론 테스트가 돌아가게 하기위해 bookRepository은 mock 객체이고, when을 사용하여 behaviour을 정의하고 합치는 `collaborator`는 필요하다.

`Verify behaviour`는 `mutation`에서도 적용된다. 물론 그러한 메서드는 void 타입을 반환을 할 수 있다. 유일한 차이는 결과가 아래에서 나온다는 점이다.

```java
public void createBook(Book book) {
	bookRepository.createBook(book);
}
...
@Test
public void insertBook() {
	BookRepository bookRepository = mock(BookRepository.class);
	BookService bookService = new BookService(bookRepository);
	Book the_dark_tower = new Book("The Dark Tower");
	bookService.createBook(the_dark_tower);

	verify(bookRepository, times(1)).createBook(the_dark_tower);
}
```

mutation의 결과를 반환하는 경우도 있다. 그럴 경우 반환값과 호출 여부를 모두 확인해야한다. 하지만 보통 이런 메서드는 사이드 이펙트를 부르는 안티패턴이고, 두 개의 메서드로 분리되어야한다.

---

**포스트 후기**

첫번째 포스트와 비슷하게 다른 메서드 호출이 중요한 메서드의 테스트는 호출여부를 확인하였다.

<br/>

### **[Unit testing behaviours without coupling to implementation details](https://softwareengineering.stackexchange.com/questions/234024/unit-testing-behaviours-without-coupling-to-implementation-details)**

Q : `save X to some data source`와 같이 일반적인 `service-repotisoty`구조에서 구현을 함께 테스트하지 않고 어떻게 테스트를 하는가?

A1)

`save X to some data source` 과 같은 경우는 외부 의존성과 통신하는 경우이므로, 특정한 메서드가 제대로 호출되었는지 테스트 해야하는 경우에 속한다. **테스트해야할 behaviour가 통신이 예상대로 이뤄졌는가**이기 때문이다.

어플리케이션과 외부 의존성 사이의 인터페이스는 구현상세가 아니고 시스템 아키텍쳐 정의이다. 변경될 일이 적다는 의미이다. 따라서 repository 인터페이스를 테스트하는 것은 그리 많은 문제를 내지 않는다.(만약 문제를 많이 내면, 인터페이스가 어플리케이션의 역할을 빼았은 경우에 속한다.)

UI, 데이터베이스, 외부 서비스를 제외한 어플리케이션의 비즈니스 로직만이 테스트와 구현상세를 분리해야하는 부분이다.

A2)

-  모든 인터페이스는 input, output, collaborator(like database) 와 함께 구성된다.
-  input 인터페이스는 테스트하라 : 메서드를 부르고, 반환값을 assert하라
-  output 인터페이스는 mock 하라 : 예상되는 메서드가 올바른 값으로 호출되었는지 확인하라
-  collaborator는 fake하라 : 간단하지만 동작하는 구현을 붙여라

database는 일반적으로 collaborator다. mock되기보단 fake를 붙이는 게 좋다. 손으로 구현하긴 어렵지만, 사용할 수 있는 도구들이 있다. 로그처럼 한번 써지고 다시는 읽히지 않은 database는 output이므로 mock되어야한다.

Input-output-collaborator에 대해 자세히

-  interface는 외부 세계(db, socket, http 등)과 통하는 port를 감싸고 있다.
-  output interface : exception을 제외한 반환값을 내지 않고, port를 통해 외부 세계로 나가는 인터페이스이다
-  input interface : 반환 값을 내며, 외부로 나가지 않는 인터페이스이다
-  Collaborator : input, output 둘 다한다

---

**후기**

답변 1에서는 service의 behaviour가 repository와 통신하는 것이므로 메서드 호출 여부를 확인해야하며, repository는 제대로 설계되었다면 변경될 일이 적으므로 괜찮다는 말이다.

답변 2는 input-output-collaborator로 인터페이스를 나눠서 repository와 같은 collaborator는 가짜 구현을 붙여 결과를 테스트 하라고 하였다.

첫번째 답변에서는 repository를 크게 변경하여 테스트가 너무 많이 깨진다면 애초에 설계가 잘못된 것이라는 점을 알았다. 두번째 답변에서는 가짜 데이터베이스를 붙여 결과를 확인하는 것도 가능하는 것을 알았다.

<br/>

## **결론**

세가지 글을 읽고 나니 구현 검증 없이 테스트를 어떻게 해야하는 것인지 어느정도 감이 잡혔다. 다음과 같은 유형으로 정리할 수 있을 거 같다

-  주어진 input에 올바른 output을 반환하는 것이 목적인 경우

   -> input 값에 따른 output이 올바른지 테스트

-  다른 메서드를 올바르게 호출하는 것이 목적인 경우

   -  호출해야하는 메서드를 올바른 값으로 호출하였는지 테스트 (행동 검증 == 구현 검증)
   -  이때 호출해야하는 것이 collaborator 인터페이스라면, fake 구현을 붙일 수도 있다.

중요한 것은 테스트 대상의 목적을 명확히 하는 것이다. 그 후 그 목적을 테스트한다면, 구현을 검증하더라도 행동을 검증하는 것이 된다.

그리고 좀 더 세부적으로 일반적인 `service-repository` 구조에서 repository가 제대로 호출되었는지 확인하는 것은 구현을 테스트하는 것이 아니라고 볼수도 있다. repository는 외부 데이터베이스의 인터페이스이므로 구현보다는 시스템 아키텍쳐에 가깝기 때문이다. 따라서 respository의 변경은 일반적으로 테스트를 깨트리지 않고, 만약 테스트를 너무 많이 깨트린다면 repository 설계가 잘못된 것이기에 마땅히 테스트를 수정해야한다.

`service-repository` 구조에서는 repository를 제대로 호출하는 것으로 검증할수도 있지만, 만약 정말로 데이터가 제대로 저장되는지 확인하고 싶다면 테스트용 DB를 사용하는 방법도 있다.

<br/>

## 주의할 점

테스트를 작성하는 것은 완벽히 하기 위해 하는 것이 아니다. **코드를 조금 더 좋게** 하기 위해 하는 것이다. 따라서 좋은 테스트를 작성하도록 노력은 하되 너무 집착하여 완벽한 테스트를 작성하고자 하진 않아도 된다.

<br/>

## 출처

-  https://jojoldu.tistory.com/614
-  https://understandlegacycode.com/blog/is-it-ok-to-change-code-for-testing-sake/
-  https://launchscout.com/blog/testing-behavior-vs-testing-implementation
-  https://craftsmen.nl/test-behaviour-not-implementation/
-  https://softwareengineering.stackexchange.com/questions/234024/unit-testing-behaviours-without-coupling-to-implementation-details

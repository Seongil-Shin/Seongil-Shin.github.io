---
title: SpEL - Spring Expression Language
author: 신성일
date: 2022-11-14 23:19:26 +0900
categories: [study, spring]
tags: [spring, spel]
---

이 문서는 개인적인 목적이나 배포하기 위해서 복사할 수 있다. 출력물이든 디지털 문서든 각 복사본에 어떤 비용도 청구할 수 없고 모든 복사본에는 이 카피라이트 문구가 있어야 한다. ([출처](https://blog.outsider.ne.kr/837))

<Br/>

# SpEL - Spring Expression Language

객체를 조회하고 조작하는 기능을 제공하며, 메서드 호출, 물자열 템플릿 기능 등의 여러가지 추가기능을 제공하는 표현식 언어이다. OGNL 등 자바에서 사용가능한 여러 EL이 있지만, SpEL은 Spring 프로젝트 전반에 걸쳐 사용하기 위해 만들어졌으며 스프링 3.0부터 지원한다.

## SpEL 표기법

```java
#{SpEL 표현식}
```

-  `#{}` 안의 표현식을 evaluation 한다.
   -  스프링에서 사용되는 `${}` 은 SpEL이 아니라 프로퍼티를 참조할 때 사용하는 표기이다. SpEL은 기본적으로 `#{}`으로 표기한다

<br/>

## SpEL 파싱

### Expression을 이용한 SpEL 파싱

```java
ExpressionParser parser = new SpelExpressionParser();
Expression expression = parser.parseExpression("1+1");
Object value1 = expression.getValue();
System.out.println(value1);    // 2

int value2 = expression.getValue(Integer.class);
System.out.println(value2);  // 2
```

-  ExpressionParser의 구현체인 SpelExpressionParser로 SpEL의 내용을 파싱하고, Expression의 getValue() 메서드를 사용해 파싱된 결과물을 Object 타입으로 얻을 수 있다.
-  getValue() 메서드에 클래스를 넣으면 타입 캐스팅도 가능하다.

### EvaluationContext를 이용한 SpEL 파싱

단순 표현식 파싱이 아닌 객체 정보들의 context가 필요한 경우 EvaluationContext를 사용할 수 있다.

```java
// name, nationality를 파라미터로 갖는 생성자
Inventor tesla = new Inventor("Nikola Tesla","Serbian");

ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("name"); // name 프로퍼티

// Context에 tesla객체를 넣어준다.
EvaluationContext context = new StandardEvaluationContext(tesla);
String name1 = (String) exp.getValue(context); //name = "Nikola Tesla"
System.out.println(name1);  // Nikola Tesla

// getValue 메서드 호출 시 StandardEvaluationContext를 사용하지 않고 객체를 직접 지정
String name2 = (String) exp.getValue(tesla);
System.out.println(name2);  // Nikola Tesla
```

-  StandardEvaluationContext에 name 프로퍼티가 평가될 객체를 지정한다.
-  두번째 방법처럼 StandardEvaluationContext를 사용하지 않고 getValue()에 직접 객체를 지정할 수 있다. 하지만 StandardEvaluationContext를 사용하면 객체 생성 비용은 들지만, 필드에 대해 캐싱하기 때문에 반복적으로 사용하면 표현식 파싱이 더 빠르다는 장점이 있다.

<br/>

## SpEL 기능 개요

### 리터럴 표현식

```java
String helloWorld = (String) parser.parseExpression("'Hello World'").getValue();
double avogadrosNumber  = (Double) parser.parseExpression("6.0221415E+23").getValue();
int maxValue = (Integer) parser.parseExpression("0x7FFFFFFF").getValue();
```

### 프로퍼티, 배열, 리스트, 맵에 대한 접근

```java
int year = (Integer) parser.parseExpression("Birthdate.Year + 1900").getValue(context);

String name = parser.parseExpression("Members[0].Name")
  .getValue(societyContext, String.class);

// Officer의 딕션어리(맵 접근)
Inventor pupin = parser.parseExpression("Officers['president']")
  .getValue(societyContext, Inventor.class);

// 값을 설정한다
parser.parseExpression("Officers['advisors'][0].PlaceOfBirth.Country")
  .setValue(societyContext, "Croatia");

// 안전한 탐색 연산자
city = parser.parseExpression("PlaceOfBirth?.City").getValue(context, String.class);
```

-  javascript의 `optional chaining` 처럼 null이 아닌지 확인하며 프로퍼티에 접근하는 문법도 사용가능하다.

### 메서드 호출

```java
String c = parser.parseExpression("'abc'.substring(2, 3)").getValue(String.class);
boolean isMember = parser.parseExpression("isMember('Mihajlo Pupin')")
  .getValue(societyContext, Boolean.class);
```

-  메서드 호출을 지원하며, 가변인자도 지원한다.

### 관계 연산자

```java
boolean trueValue = parser.parseExpression("2 == 2").getValue(Boolean.class);
boolean falseValue = parser.parseExpression("2 < -5.0").getValue(Boolean.class);
boolean falseValue = parser.parseExpression("'xyz' instanceof T(int)")
  .getValue(Boolean.class);
// 정규표현식과 비교하기 위해서 mathes 키워드를 사용한다.
boolean trueValue = parser.parseExpression("'5.00' matches '^-?\\d+(\\.\\d{2})?$'")
  .getValue(Boolean.class);
```

### 논리 연산자

```java
boolean falseValue = parser.parseExpression("true and false").getValue(Boolean.class);
boolean trueValue = parser.parseExpression("true or false").getValue(Boolean.class);
boolean falseValue = parser.parseExpression("!true").getValue(Boolean.class);
```

### 수식 연산자

```java
int two = parser.parseExpression("1 + 1").getValue(Integer.class); // 2
int four =  parser.parseExpression("1 - -3").getValue(Integer.class); // 4
int six =  parser.parseExpression("-2 * -3").getValue(Integer.class); // 6
int minusTwo =  parser.parseExpression("6 / -3").getValue(Integer.class); // -2
int three =  parser.parseExpression("7 % 4").getValue(Integer.class); // 3
int minusTwentyOne = parser.parseExpression("1+2-3*8").getValue(Integer.class); // -21
```

### 할당

```java
Inventor inventor = new Inventor();
StandardEvaluationContext inventorContext = new StandardEvaluationContext(inventor);

parser.parseExpression("Name").setValue(inventorContext, "Alexander Seovic2");

// 대신에 getValue()로도 할당이 가능하다.
String aleks = parser.parseExpression("Name = 'Alexandar Seovic'")
  .getValue(inventorContext, String.class);
```

### 생성자 호출

```java
Inventor einstein = p.parseExpression(
  "new org.spring.samples.spel.inventor.Inventor('Albert Einstein', 'German')")
  .getValue(Inventor.class);

//리스트의 add 메서드내에서 새로운 inventor 인스턴스를 생성한다
p.parseExpression(
  "Members.add(new org.spring.samples.spel.inventor.Inventor('Albert Einstein', 'German'))")
  .getValue(societyContext);
```

-  생성자를 새로 만들 수도 있다. primitive 타입과 String 외에는 모두 정규화된 클래스명을 사용해야한다.

### 타입

```java
Class dateClass = parser.parseExpression("T(java.util.Date)").getValue(Class.class);

Class stringClass = parser.parseExpression("T(String)").getValue(Class.class);

boolean
 trueValue = parser.parseExpression(
  "T(java.math.RoundingMode).CEILING < T(java.math.RoundingMode).FLOOR")
  .getValue(Boolean.class);
```

-  `T` 연산자를 클래스 인스턴스를 지정하는데 사용할 수 있다. 정적 메서드도 이 연산자를 사용해서 호출할 수 있다.
-  StandardEvaluationContext는 타입을 찾으려고 TypeLocator를 사용하고 StandardTypeLocator(교체할 수 있다)는 java.lang 패키지로 만들어진다. 즉, java.lang 내에서 타입을 참조하는 T()는 정규화될 필요는 없지만 다른 모든 타입참조는 정규화되어야 한다.

### 빈(Bean) 참조

```java
ExpressionParser parser = new SpelExpressionParser();
StandardEvaluationContext context = new StandardEvaluationContext();
context.setBeanResolver(new MyBeanResolver());

// 평가하는 동안 MyBeanResolver에서 resolve(context,"foo")를 호출할 것이다
Object bean = parser.parseExpression("@foo").getValue(context);
```

-  context가 빈 리졸버로 설정되어있다면 @ 기호를 사용해서 표현식에서 빈을 검색하는 것이 가능하다.

### 배열 생성

```java
int[] numbers1 = (int[]) parser.parseExpression("new int[4]").getValue(context);
int[] numbers2 = (int[]) parser.parseExpression("new int[]{1,2,3}").getValue(context);
int[][] numbers3 = (int[][]) parser.parseExpression("new int[4][5]").getValue(context);
```

### 인라인 리스트

```java
// 4개의 숫자를 담고 있는 자바 리스트로 평가된다
List numbers = (List) parser.parseExpression("{1,2,3,4}").getValue(context);
```

### 삼항 연산자

```java
String falseString = parser.parseExpression("false ? 'trueExp' : 'falseExp'")
  .getValue(String.class);

// 엘비스 문법 a ? a : b --> a ?: b
String name = parser.parseExpression("null?:'Unknown'").getValue(String.class);
```

-  삼항연산자 사용이 가능하고, 단축형인 엘비스 문법도 사용할 수 있다.

### 변수

```java
Inventor tesla = new Inventor("Nikola Tesla", "Serbian");
StandardEvaluationContext context = new StandardEvaluationContext(tesla);
context.setVariable("newName", "Mike Tesla");

parser.parseExpression("Name = #newName").getValue(context);

System.out.println(tesla.getName()) // "Mike Tesla"
```

-  context에 변수를 선언하여 표현식 내에서 변수를 참조할 수 있다.
-  다만 `#this`와 `#root` 변수는 항상 정의되어있다. 각각 현재 평가객체, 루트 컨텍스트 객체를 참조한다.

### 사용자 정의 함수

```java
ExpressionParser parser = new SpelExpressionParser();
StandardEvaluationContext context = new StandardEvaluationContext();

context.registerFunction("reverseString",
                         StringUtils.class.getDeclaredMethod("reverseString",
                                                             new Class[] { String.class }));
String helloWorldReversed =
          parser.parseExpression("#reverseString('hello')").getValue(context, String.class);
```

-  context에 함수를 등록해서 SpEL을 확장할 수 있다.

### 컬렉션 투영(Collection projection)

```java
List placesOfBirth = (List)parser.parseExpression("Members.![placeOfBirth.city]");
```

-  하위 표현식을 평가해서 새로운 컬렉션을 반환한다. 컬렉션의 특정 필드만으로 리스트를 만들고 싶을 떄 사용한다.
-  맵에서도 투영기능을 사용할 수 있고 맵의 경우 투영 표현식은 맵의 각 엔트리마다(자바 Map.Entry로 표현되는) 평가된다. 맵의 투영결과는 맵의 각 엔트리에 대한 투영 표현식의 평가결과로 이루어진 리스트이다

### 컬렉션 선택

```java
List<Inventor> list = (List<Inventor>)parser.parseExpression(
  "Members.?[Nationality == 'Serbian']")
  .getValue(societyContext);

Map newMap = parser.parseExpression("map.?[value<27]").getValue();
```

-  컬렉션을 필터링해서 원래 요소의 서브셋을 가진 새로운 컬렉션을 반환한다.

### 표현식 템플릿

```java
String randomPhrase =
   parser.parseExpression("random number is #{T(java.lang.Math).random()}",
                          new TemplateParserContext()).getValue(String.class);
```

-  `#{}`로 구분하여 하나 이상의 평가 블럭을 가진 리터럴 문자를 섞을 수 있다.

<br/>

## 사용예시

### @Value 애노테이션에서 SpEL 사용

```java
@Value("#{1+1}")
int value;

@Value("#{'hello ' + 'world'}")
String greeting;

@Value("#{1 eq 5}")
boolean trueOrFalse;

@Value("Literal String")
String literalString;

@Override
public void run(ApplicationArguments args) throws Exception {
    System.out.println(value);					// 2
    System.out.println(greeting);				// hello world
    System.out.println(trueOrFalse);		// false
    System.out.println(literalString);	// Literal String
}
```

-  빈이 만들어질 때, @Value() 안의 값이 #{} 표기로 감싸져 있으면, SpEL로 파싱하고 평가해서 결과값을 변수에 할당함

### SpEL과 프로퍼티

```properties
my.value=100
```

```java
@Value("#{'${my.value}' eq '100'}")
boolean isEqual;

@Override
public void run(ApplicationArguments args) throws Exception {
    System.out.println(isEqual);		// true
}
```

### 빈 참조

```java
@Component
public class Sample {
  private int value = 123;

  public int getValue() {
    return value;
  }
}
```

```java
@Value("#{sample.Value}")
int sampleValue;
```

## 참고

SpEL도 해당하는 타입으로 변환할 때 ConversionService를 사용한다

<br/>

## 출처

-  https://blog.outsider.ne.kr/835
-  https://blog.outsider.ne.kr/837
-  https://atoz-develop.tistory.com/entry/Spring-SpEL-Spring-Expression-Language

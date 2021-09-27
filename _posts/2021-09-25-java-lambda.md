---
title: Java 람다(Lambda)
date: 2021-09-25 02:29:30
categories: [Java]
tags: [java, lambda]     # TAG names should always be lowercase
---
# 람다식
자바 5(JDK 1.5)에 추가된 제너릭 이후 가장 큰 변화는 바로 자바 8(JDK 1.8)에 추가된 람다식이다
이를 통해 객체지향 언어인 JAVA가 함수형 언어의 특징을 가지게 되었다

### 람다식이란?
람다식이란 간단히 메서드를 하나의 식으로 표현한 것이다
메서드를 람다식으로 표현하면 메서드의 이름과 반환값이 없어지므로, 람다식을 **익명 함수**라고도 한다

```java
int[] arr = new arr[5];
Arrays.setAll(arr, i -> (int)((Math.random()*5)+1));    // 1~5 사이의 정수 5개 저장
```
<br><br>

### 람다식 작성하기
람다식은 메서드의 이름과 반환타입을 제거하고 매개변수 선언부와 몸통 사이에 **->** 를 추가한다
반환값이 있는 메서드의 경우 return문 대신 식(expression으로 대체할 수 있다)
```java
(int a, int b) -> a > b ? a : b
```
또한 매개변수의 타입을 예상할 수 있는 경우 타입의 생략이 가능하다
```java
(a, b) -> a > b ? a : b
```
선언된 매개변수가 하나일 경우 괄호도 생략할 수 있다
```java
a -> a>0 ? true : false
```
마찬가지로 함수 몸통부의 문장이 하나일 경우 중괄호도 생략이 가능하다

<br><br>
### 함수형 인터페이스
자바에서 모든 메서드는 클래스에 포함되어야 한다
그렇다면 람다는 어떤 클래스에 포함될까?
결론부터 말하면 **Object를 구현하는 익명 클래스**라고 생각하면 된다
함수의 파라미터로 람다식을 요구하는 경우를 많이 보았을 것이다
람다식을 받는 클래스를 타고 들어가면 @FunctionalInterface 어노테이션이 붙은 인터페이스를 발견할 수 있다
이 인터페이스는 하나의 추상 메서드만 가지는데, 우리는 이를 구현하는 것이다
```java
@FunctionalInterface
interface MyFunction(){
    void myMethod();
}
```
람다식을 요구하는 메서드를 작성해 보자
```java
public void requireLambda(MyFunction f){
    f.myMethod();
}

MyFunction myFunction = () -> System.out.println("myFunction");
requireLambda(myFunction);
```
위를 보면 참조변수로 메서드를 주고받는 것 같은 효과를 낼 수 있다
하지만 실제로는 익명 클래스의 객체를 주고받는 것을 기억하자

### java.util.function패키지
일반적으로 많이 사용되는 형식의 메서드는 java.util.function패키지에 함수형 인터페이스로 정의해 놓았다
|함수형 인터페이스|메서드|설명|
|---|---|---|
|java.lang.Runnable|void run()|매개변수도 없고 반환값도 없음|
|Supplier<T>|T get()|매개변수는없고 반환값만 있음|
|Consumer<T>|void accept(T t)|Supplier와 반대로 반환값이 없고 매개변수만 있음|
|Function<T,R>|R apply(T t)|하나의 매개변수를 받아 하나를 return함|
|Predicate<T>|boolean test(T t)|하나의 매개변수를 받아 boolean을 return함|

### 메서드 참조
하나의 메서드만 호출하는 람다식은 **'클래스이름::메서드이름'** 또는 **'참조변수::메서드이름'** 으로 바꿀 수 있다


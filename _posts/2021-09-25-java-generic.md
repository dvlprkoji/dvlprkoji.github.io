---
title: Java 제너릭스(Generics)
date: 2021-09-25 02:29:30
categories: [Java]
tags: [java, comparator, comparable]     # TAG names should always be lowercase
---

#제너릭스란?
제너릭스란 다양한 타입의 객체들을 다루는 메서드나 컬렉션 클래스에 컴파일 시의 타입 체크를 해주는 기능이다   
객체의 타입을 컴파일 시 체크하기 때문에 객체의 타입 안정성을 높이고 형변환의 번거로움이 줄어든다   

```java
제너릭스의 장점
1. 타입 안정성을 제공한다
2. 타입체크와 형변환을 생략할 수 있으므로 코드가 간결해 진다
```
<br><br>
### 제너릭 클래스의 선언
제너릭 타입은 클래스와 메서드에 선언할 수 있는데, 먼저 클래스에 선언하는 제너릭 타입을 알아보자   
예를 들어 Box가 다음과 같이 정의되어 있다고 가정하자
```java
class Box{
    Object item;

    void setItem(Object item) {
        this.item = item;
    }
    Object getItem() {
        return item;
    }
}
```
이 클래스를 제너릭으로 선언하면 다음과 같다
```java
class Box<T>{
    T item;

    void setItem(T item) {
        this.item = item;
    }
    T getItem() {
        return item;
    }
}
```
<br><br>

### 제너릭스의 용어
제너릭스에서 사용되는 용어들은 헷갈리기 쉽다. 한번 정리하고 넘어가겠다  
```java
class Box<T> { }
Box     : 원시 타입
T       : 타입 변수
Box<T>  : 제너릭 클래스, T Box라고 읽는다
new Box<T> : 제너릭 타입 호출
```
<br><br>

### 제너릭스의 제한

1. 제너릭 클래스의 객체를 생성할 때, 객체별로 다른 타입을 가질 수 있지만 클래스 변수는 제너릭 타입을 가질 수 없다  
클래스 변수는 인스턴스 변수와 무관하게 같은 값을 가져야 하기 때문이다
```java
class Box<T>{
    static T item;                          // 에러
    static int compare(T t1, T t2){...};    // 에러
}
```
2. 제너릭 타입의 배열을 생성하는 것도 허용되지 않는다  
이는 new 연산자 때문인데, 이 연산자는 컴파일 타임에 그 타입을 정확히 알야야 하기 때문이다
```java
class Box<T>{
    T[] itemArr;                                // OK
    T[] toArray(){
        T[] tmpArr = new T[itemArr.length];     // 에러. 제네릭 배열 생성 불가
        return tmpArr;
    }
}
```
<br><br>

### 제너릭 클래스의 객체 생성과 사용
참조변수와 매개변수에 대입된 타입은 같아야 한다  
두 타입이 상속 관계에 있다고 해도 동일하다
```java
Box<Apple> appleBox = new Box<Grape>();         // 에러
Box<Fruit> fruitBox = new Box<Apple>();         // 에러
```
단 두 제너릭 클래스가 상속 관계에 있고, 매개변수의 타입이 동일한 경우 대입이 가능하다
```java
Box<Apple> appleBox = new FruitBox<Apple>();    // OK
```
생성된 제너릭 클래스의 인스턴스로 자손 관계의 객체는 대입이 가능하다
```java
Box<Fruit> fruitBox = new Box<Fruit>;
fruitBox.add(new Apple());
```
<br><br>

### 제한된 제너릭 클래스
제너릭 클래스는 하나의 타입만 지정하여 받을 수 있다  
하지만 제너릭 타입에 extends를 사용하면 특정 타입의 자손들만 대입할 수 있다
```java
class FruitBox<T extends Fruit>{}       // Fruit의 자손들만 대입 가능
```
Fruit의 자손이면서 Eatable 인터페이스를 구현해야 하면 다음과 같이 &기호를 사용하면 된다
```java
class FruitBox<T extends Fruit & Eatable>{}
```
<br><br>

### 와일드 카드
클래스 메서드의 매개변수에는 제너릭 타입 매개변수를 지정할 수 없다  
따라서 특정 타입을 지정해 주어야 사용할 수 있는데 이 경우 상속 관계의 클래스들도 모두 정의해야 하는 문제점이 있다
```java
static Juice makeJuice(FruitBox<Fruit> box){...}
static Juice makeJuice(FruitBox<Apple> box){...}
```
하지만 위와 같이 정의해도 메서드 중복 정의 에러가 발생한다  
이러한 상황에 사용할 수 있는 것이 와일드 카드(?)이다
```java
<? extends T>   와일드 카드의 상한 제한
<? super T>     와일드 카드의 하한 제한
<?>             제한 없음 (= <? extends Object>)
```
이를 적용하면 위 코드를 다음과 같이 바꿀 수 있다
```java
static Juice makeJuice(FruitBox<? extends Fruit> box){...}
```
<br><br>

### 제너릭 타입의 형변환
제너릭 타입과 원시 타입의 형변환은 가능할까?
```java
Box     box = null;
Box<T>  boxT = null;
boxT    = (Box<T>)box;  // OK
box     = (Box)boxT;    // OK
```
경고가 발생하긴 하지만 형변환이 가능하다  
그렇다면 대입된 타입이 다른 두 제너릭 타입끼리의 형변환은 가능할까?
```java
Box<Fruit> fruitBox = null;
Box<Grape> grapeBox = null;
fruitBox = (Box<fruit>)grapeBox;    // Error
grapeBox = (Box<Grape>)fruitBox;    // Error
```
불가능하다. 대입된 타입이 조상 타입이어도 불가능하다  
그렇다면 Box<? extends Fruit>은 어떨까?
```java
Box<? extends Fruit> fruitBox = null;
fruitBox = new Box<Apple>();    // OK
fruitBox = new Box<Grape>();    // OK

Box<Apple> appleBox = null;
appleBox = (Box<Apple>)fruitBox; // OK
```
양방향 변환 모두 가능하다  
그렇다면 와일드카드가 사용된 제너릭 타입끼리는 형변환이 가능할까?
```java
Box<? extends String> stringBox = null;         // OK
Box<? extends Object> objectBox = null;         // OK
objectBox = (Box<? extends Object>)stringBox;   // OK
stringBox = (Box<? extends String>)objectBox;   // OK
```
모두 가능하다
<br><br>

### 제너릭 타입의 제거
컴파일러는 제너릭 타입을 이용해 소스파일을 체크해 컴파일 시점에 각 타입을 변환한다  
따라서 class 파일에는 제너릭 타입이 존재하지 않는데 이는 제너릭 이전의 소스 코드와 호환성을 위한 것이다


---
title: Java 다형성(Polymorphism)
date: 2021-09-27 00:00:01
categories: [Java]
tags: [java, polymorphism]     # TAG names should always be lowercase
---

### 다형성이란?
조상클래스 타입의 참조변수로 자손클래스의 인스턴스를 참조할 수 있게 하는 것이다  
Unit을 상속받는 Marin 클래스가 있다고 가정하자
```java
public class Unit{
    public int hp;
    public int mp;
}
public class Marin extends Unit {
    public void attack(){...}
}
```
Unit 참조 변수는 Marin 객체를 가리킬 수 있을까?
```java
Unit unit = new Marin();        // OK
```
그렇다. 가리킬 수 있지만 unit을 통해서는 Maring의 멤버들을 사용할 수 없는 것을 주의한다  
그렇다면 반대로 Marin 참조 변수는 Unit 객체를 가리킬 수 있을까?
```java
Marin marin = new Unit();       // Error
```
불가능하다. Marin에 정의된 멤버들은 Unit객체를 통해 접근할 수 없기 때문이다

### 참조변수의 형변환
형변환은 참조변수의 타입을 변경하는 것을 말한다  
여기서 대입된 인스턴스에는 아무런 영향을 미치지 않는다는 것을 이해해야 한다  
단지 참조변수의 형변환을 통해 참조하고 있는 인스턴스에서 사용할 수 있는 멤버의 개수를 조절할 뿐이다  
<br><br>
참조변수의 형변환은 () 기호를 통해 적용할 수 있다  
다운 캐스팅을 할 때 이를 생략할 수 있는 것이고  
위의 예시는 사실 다음과 같은 것이다
```java
Unit unit = (Unit)new Marin();
```
그렇다면 업 캐스팅은 생략이 불가능하다는 것인데 업 캐스팅을 해보면 어떨까?
```java
Marin marin = (Marin)new Unit();        // OK
```
컴파일 타임 오류는 발생하지 않는다. 하지만 실행할 경우 ClassCastException이 발생한다  
원칙상 조상 타입과 자손 타입끼리 형변환이 가능하지만 조상 객체를 다운 캐스팅하면 자손의 멤버를 사용할 수 없기 때문에 에러가 발생했다

### instanceof 연산자
참조변수가 참조하고있는 인스턴스의 실제 타입을 알아보려면 instanceof 연산자를 사용하면 된다  
조상 타입의 멤버 메서드 중 자손 타입을 사용하는 메서드가 있다면 형변환을 통해 사용하자
```java
public class Unit{}
public class Marin extends Unit{...}
public class Tank extends Unit{...}

public doSomething(Unit unit){
    if(unit instanceof Marin){
        Marin marin = (Marin)unit;
        ...
    }
    else if(unit instanceof Tank){
        Tank tank = (Tank)unit;
        ...
    }
}
```
### 참조변수와 인스턴스의 연결
조상 클래스와 자손 클래스에 동일한 이름의 멤버를 정의했을 때, 호출하면 어떤 결과가 나올까?  
이는 멤버의 종류와 참조하는 인스턴스의 종류에 따라 다르다  
결론부터 말하면, **멤버 변수**의 경우 **참조 변수**의 타입을 따라가고, **멤버 함수**는 참조하는 **인스턴스**의 타입을 따라간다

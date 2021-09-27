---
title: Java 내부 클래스(Inner Class)
date: 2021-09-27 00:00:01
categories: [Java]
tags: [java, inner class]     # TAG names should always be lowercase
---
### 내부 클래스란?
내부 클래스는 클래스 내부에 선언된 클래스이다  
내부 클래스를 사용하는 경우 내부 클래스에서 외부 클래스의 멤버들을 쉽게 접근할 수 있으며 코드 복잡성이 줄어든다는 장점이 있다


### 내부 클래스의 종류와 특징
|내부 클래스|특징|
|---|---|
|인스턴스 클래스<br>(instance class)|외부 클래스의 멤버 변수들과 같은 위치에 선언한다. 주로 외부 클래스의 인스턴스멤버들과 관련된 작업에 사용될 목적으로 선언된다|
|스태틱 클래스<br>(static class)|외부 클래스의 멤버변수 선언위치에 선언하며, 외부 클래스의 static멤버처럼 다루어진다. 주로 외부 클래스의 static멤버, 특히 static 메서드에 사용될 목적으로 선언된다|
|지역 클래스<br>(local class)|외부 클래스의 메서드나 초기화블럭 안에 선언하며, 선언 영역에서만 사용될 수 있다|
|익명 클래스<br>(anonymous class)|클래스의 선언과 생성을 동시에 하는 이름없는 클래스|

```java
public class OuterClass{
    class InstanceClass{}                   // 인스턴스 클래스
    static class staticClass{}              // 스태틱 클래스
    void outerMethod(){
        class localClass{}                  // 지역 클래스
    }
}
```

### 익명 클래스
익명 클래스는 이름이 없다. 클래스의 선언과 생성을 동시에 하기 때문에 재사용이 불가능한 일회용 클래스이다  
이름이 없기 때문에 생성자도 없으며, 하나의 클래스를 상속받거나 하나의 인터페이스를 구현할 수 있다
```java
abstract class Unit{
    void doSomething();
}
Unit unit = new Unit() {
    void doSomething(){...}
}
```

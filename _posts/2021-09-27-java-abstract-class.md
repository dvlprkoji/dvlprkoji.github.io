---
title: Java 추상 클래스(Abstract Class), 인터페이스(Interface)
date: 2021-09-27 00:00:01
categories: [Java]
tags: [java, abstract class, abstract method]     # TAG names should always be lowercase
---
# 추상 클래스(Abstract Class)

### 추상 클래스란?
추상 클래스는 추상화 정도가 일반 클래스와 인터페이스의 사이에 있는 클래스이다  
추상 클래스는 **추상 메서드를 포함**하고 있다는 것을 제외하고 일반 클래스와 전혀 다르지 않다  
추상 클래스는 단독으로 인스턴스를 생성할 수 없으며 추상 클래스를 상속하여 추상 메서드를 정의해야 사용할 수 있다

### 추상 메서드(Abstract Method)
메서드의 선언부만 작성하고 구현부는 작성하지 않은 채로 남겨 둔 것이 추상 메서드이다  
일반적으로 상속받을 클래스에 따라 메서드의 내용이 달라질 수 있기 때문에 주석을 통해 그 기능만 지정해 놓는다
```java
/* to calculate sum of stream type */
abstract int calcSum(Stream<?> stream);
```
추상 클래스를 상속받는 메서드는 추상 메서드를 모두 구현해야 한다  
만약 구현하지 않은 메서드가 존재한다면 자식 클래스도 추상 클래스로 지정해야 한다

### 추상 클래스의 작성
상속이 자손 클래스를 만드는데 조상 클래스를 사용하는 것이라고 하면,  
추상화는 기존 클래스의 공통부분만 뽑아내어 조상 클래스를 만드는 것이라고 할 수 있다
```java
추상화 - 클래스간의 공통점을 찾아내어 공통의 조상을 만드는 작업
구체화 - 상속을 통해 클래스를 구현, 확장하는 작업
```

### 추상 클래스의 의미
어차피 자손 클래스에서 오버라이딩하여 새로운 함수를 작성할 것이기 때문에 추상 클래스가 필요없어 보일 수 있다  
하지만 추상 클래스를 지정함으로서 추상 메서드의 오버라이딩을 강제할 수 있다는 장점이 있다

# 인터페이스(Interface)

### 인터페이스란?
인터페이스는 일종의 추상클래스이다    
인터페이스는 추상클래스처럼 추상메서드를 갖지만 몸통을 갖춘 메서드를 가질 수 없다  
인터페이스가 가질 수 있는 것은 오직 **추상메서드**와 **상수**뿐이다

### 인터페이스의 작성
인터페이스의 작성법은 다음과 같다
```java
interface 인터페이스 이름{
    public static final 타입 상수이름 = 값;
    public abstract 메서드명(매개변수목록);
}
```
인터페이스 작성은 클래스 작성법과 유사하나 몇가지 제한사항이 있다
```java
1. 모든 멤버변수는 public static final이어야 하며, 이는 생략이 가능하다
2. 모든 메서드는 public abstract 이어야 하며 이는 생략할 수 있다
```
따라서 일반적으로 작성하는 인터페이스는 다음과 같다
```java
interface Unit{
    int age = 0;    // public static final int age = 0;
    int getAge();   // public abstract int getAge();
}
```
원래 인터페이스의 모든 메서드는 추상메서드이어야 하는데,  
JDK 1.8부터는 static 메서드와 defualt 메서드를 추가하는 방향으로 변경되었다

### 인터페이스의 상속
인터페이스는 인터페이스만 상속받을 수 있으며 클래스와 달리 다중상속이 가능하다
```java
interface Movable{}
interface Attackable{}
interface Fightable extends Movable, Attackable{
    ...
}
```
오버라이딩 할 때는 조상의 메서드보다 넓은 범위의 접근 제어자를 지정해야 한다  
인터페이스의 추상 메서드는 모두 public abstract이므로 인터페이스를 구현할 때 모두 public으로 지정해야 한다
```java

interface Unit{
    String name;
    String getName();
}
public class Marin implements Unit{
    public String getName(){
        return this.name;
    }
}
```
### 인터페이스를 이용한 다중상속
두 조상으로부터 상속받는 멤버 중 동일한 이름을 가지거나 동일한 선언부를 가진 메서드가 존재한다고 가정해 보자  
동일한 이름을 가지기 때문에 두 조상 중 어느 조상으로부터 상속받은 것인지 알 수 없을 것이다  
따라서 자바에서는 다중 상속을 허용하지 않도록 구현해 놓았다  
하지만 인터페이스를 사용한다면 다중 상속이 불가능한 것은 아니다  
TV 클래스와 VCR 클래스를 모두 상속받는 TVCR 클래스를 만들고 싶다고 하자  
우리는 하나의 클래스를 상속받고 다른 하나는 클래스 내부에 포함시켜 내부적으로 인스턴스를 생성하도록 할 것이다
```java
public class Tv{
    protected boolean power;
    public void power(){power = !power;}
}
public class VCR{
    protected int counter;
    public void play(){...}
}
```
먼저 VCR의 상위 인터페이스를 작성해야 한다
```java
public interface IVCR{
    void play();
}
```
TV를 상속받고 VCR을 내부적으로 생성하는 TVCR클래스를 만들자
```java
public class TVCR extends TV implements IVCR{
    VCR vcr;
    public TCVR(){
      this.vcr = new VCR();
    }
    public void play(){
        vcr.play();
    }
}
```

### 디폴트 메서드와 static 메서드
원래 인터페이스에는 추상 메서드만 선언할 수 있었는데, JDK 1.8부터 디폴트 메서드와 static 메서드가 추가되었다

#### 디폴트 메서드
클래스에 새로운 메서드를 작성하는 것은 간단하지만, 인터페이스에 메서드를 추가하는 것은 매우 복잡하다  
이런 경우 디폴트 메서드를 사용하면 해결할 수 있다
```java
public interface Interface{
    default void defaultMethod(){...};
}
```

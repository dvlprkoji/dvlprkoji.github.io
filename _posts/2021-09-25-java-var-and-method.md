---
title: Java 변수(Variables)와 메서드(Method)
date: 2021-09-25 02:29:30
categories: [Java]
tags: [java, Variables, Method]     # TAG names should always be lowercase
---
<br><br>

# 선언 위치에 따른 변수의 종류
변수는 클래스변수와 인스턴스변수, 지역변수가 있다

|변수의 종류|선언위치|생성시기|생성위치|
|---|---|---|---|
|클래스변수|클래스 영역|클래스가 메모리에 올라갈 때|메서드 정의 영역|
|인스턴스변수|클래스 영역|인스턴스가 생성되었을 때|힙 영역|
|지역변수|클래스 이외의 영역|변수 선언문이 수행되었을 때|스택 영역

다음은 각 변수의 생성 예시이다
```java
public class tmpClass{
    int iv;                 // 인스턴스 변수
    static int cv;          // 클래스 변수
    public tmpClass(){
        int lv;             // 지역 변수
    }
}
```

### 클래스 변수와 인스턴스 변수
클래스 변수와 인스턴스 변수를 이해하기 위해 카드 예시를 들어보겠다
```java
public class Card{
    String type;
    int val;

    static int width = 100;
    static int height = 250;
}
```
카드의 공통적인 특성들은 static으로 선언하여 모든 인스턴스가 같은 값을 가지게 하였다

### 기본형 매개변수와 참조형 매개변수
메서드 매개변수의 타입이 **기본형**이면 **기본형 값이 복사**되고  
**참조형**이면 값이 **저장된 주소**를 읽어온다
```java
Student s = new Student("A",30);
changeIntAge(s.getAge());               // Student의 age 변경되지 않음
changeIntegerAge(s.getAge());          // Student의 age 변경됨

void changeIntAge(int age){
  age=0;
}
void changeIntegerAge(Integer age){
  age = 0;
}
```

### 인스턴스 메서드와 클래스 메서드
**인스턴스 메서드**는 메서드의 작업을 수행하는데 **인스턴스 변수를 필요로 하는** 메서드이다  
**클래스 메서드**는 **인스턴스와 관련 없는**(인스턴스 변수나 인스턴스 메서드를 사용하지 않는) 메서드이다  
다음은 인스턴스와 클래스를 사용하는 방법이다  
1. 모든 인스턴스에 공통적으로 사용해야하는 것에 static을 붙인다  
2. 클래스 변수는 인스턴스를 생성하지 않아도 사용할 수 있다
3. 클래스 메서드는 인스턴스 변수를 사용할 수 없다
4. 메서드 내에서 인스턴스 변수를 사용하지 않는다면 static을 붙이는 것을 고려한다

```java
- 클래스의 멤버변수 중 모든 인스턴스에 공통된 값을 유지하는 것이 있다면 static을 붙여준다
- 작성 메서드 중 인스턴스 변수나 인스턴스 메서드를 사용하지 않는 메서드에 static을 붙일 것을 고려한다
```

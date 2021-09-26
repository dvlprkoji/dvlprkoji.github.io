---
title: Java Comparator와 Comparable
date: 2021-09-25 02:29:30
categories: [Java]
tags: [java, comparator, comparable]     # TAG names should always be lowercase
---

# Comparator와 Comparable
두 클래스는 다음과 같은 경우에 사용한다
```java
comparator = 기본 정렬기준 외에 다른 기준으로 정렬하고자 할 때 사용
comparable = 기본 정렬기준을 구현하는데 사용
```
둘의 return 타입은 모두 int이며 비교대상이 **같은 경우 0, 비교값보다 작은 경우 음수, 비교값보다 큰 경우 양수**를 반환한다

# Comparable
Comparable 인터페이스는 다음과 같이 compareTo 함수를 구현해야 한다
```java
public interface Comparable{
  int compareTo(Object o);
}
```
# Comparator
Comparator 인터페이스는 다음과 같이 compare 함수를 구현해야 한다
```java
public interface Comparator{
  int compare(Object o1, Object o2);
}
```
다음은 Stream sorted의 파라미터에 들어갈 Comparator 메소드를 구현한 것이다
```java
stream.sorted(new Comparator(){
    @override
    public int compare(object o){
        return this.o - o;
    }
})
```



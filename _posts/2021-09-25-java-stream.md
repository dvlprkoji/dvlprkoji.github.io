---
title: Java 스트림(Stream)
date: 2021-09-25 02:29:30
categories: [Java]
tags: [java, stream]     # TAG names should always be lowercase
---

# Stream 이란?
데이터 소스의 종류와 무관하게 같은 방식으로 다룰 수 있게 메서드를 정의한 것  
스트림을 사용한 코드가 간결하고 이해하기 쉬우면서 재사용성이 높다

### Stream은 데이터 소스를 변경하지 않는다
스트림은 데이터 소스를 읽기만 할 뿐 소스를 변경하지 않는다  
필요하다면 조작한 결과를 컬렉션이나 배열에 담아 반환할 수 있다

### 스트림은 일회용이다
스트림은 Iterator처럼 일회용이다  
스트림을 한번 사용하면 닫혀서 사용할 수 없기 때문에 다시 사용하려면 스트림을 새로 생성해야 한다

```java
strStream1.sorted().forEach(System.out::println);
int numOfStr - strStream1.count(); // 에러. 스트림이 이미 닫혔음.
```

### 스트림은 작업을 내부 반복으로 처리한다
스트림의 간결함은 내부 반복으로 만들어진다  
반복문을 메서드의 내부에 숨길 수 있다  
forEach()는 스트림에 정의된 메서드 중 하나로 매개변수에 대입된 람다식을 모든 요소에 적용한다

```java
stream.forEach(System.out::println);    // 모든 요소에 대해 println
```

### 스트림의 연산
스트림이 제공하는 연산으로는 중간 연산과 최종 연산이 있다  
  - 중간 연산 : 연산 결과가 스트림인 연산, 스트림에 연속해서 중간 연산할 수 있음
  - 최종 연산 : 연산 결과가 스트림이 아닌 연산, 스트림의 요소를 소모하므로 한번만 가능

### 지연된 연산
스트림 연산에서 최종 연산이 수행되기 전까지는 중간연산이 수행되지 않는다  
최종 연산이 수행되어야 비로소 스트림 요소들이 중간 연산을 거쳐 최종 연산에서 소모된다

### 병렬 스트림

# 스트림 만들기
스트림의 소스가 될 수 있는 대상은 배열, 컬렉션 등 다양하며 이로부터 스트림을 생성하는 방법을 배울 것이다

### 컬렉션
Collection 클래스에 stream()이 정의되어 있다  
따라서 Collection의 자손들인 List와 Set을 구현한 컬렉션 클래스는 이 메서드로 스트림을 생성할 수 있다
```java
Stream<T> Collection.stream()
```
예를 들어 List로부터 스트림을 생성하는 코드는 다음과 같다
```java
List<Integer> list = Arrays.asList(1,2,3,4,5);
Stream<Integer> intStream = list.stream();
```

### 배열
배열을 소스로 하는 스트림을 생성하는 메서드는 다음과 같이 Stream과 Arrays에 정의되어 있다
```java
Stream.of(T... values)
Stream.of(T[])
Arrays.stream(T[])
Arrays.stream(T[] array, int startInclusive, int endExclusive)
```

### 람다식 - iterate(), generate()
iterate 함수는 seed로부터 람다식 f로 의해 계산된 결과를 다시 seed로 하여 계산을 반복한다
```java
Stream<Integer> evenStream = Stream.iterate(0, n->n+2);
```
generate 함수는 iterate 함수와 계산된 결과를 다시 seed로 하지 않는다는 점이 다르다

# 스트림의 중간연산

### 스트림 자르기 - skip(), limit()
skip(long n) 함수는 처음 n개의 요소를 건너뛰는 함수이다  
limit(long maxSize) 함수는 스트림의 요소를 maxSize개로 제한한다
```java
IntStream intStream = IntStream.rangeClosed(1, 10);
intStream.skip(3).limit(3).forEach(System.out::println);    // 345 출력
```

### 스트림 요소 걸러내기 - filter(), distinct()
distinct 함수는 스트림에서 중복된 요소를 제거한다  
filter 함수는 주어진 조건에 맞지 않는 요소를 걸러낸다
```java
IntStream intStream = IntStream.of(1,2,2,3,3,3,4,4,4,4,5,5,5,5,5);
intStream.distinct().forEach(System.out::println); // 1,2,3,4,5
intStream.filter(i -> i%2==0).forEach(System.out::println);     // 2,4
```

### 정렬
스트림을 정렬할 때는 sorted를 사용하면 된다.
```java
Stream<T> sorted(Comparator<? super T> comparator)
```
sorted 함수의 매개변수로는 Comparator 뿐만 아니라 람다 함수도 들어갈 수 있다  
sorted 함수의 매개변수로 Comparator를 사용할 경우 Comparator의 default 메소드와 static 메소드를 사용하면 정렬이 쉬워진다  
다음은 가장 많이 사용되는 Comparator.comparing 함수와 Comparator.thenComparing 함수의 사용 예시이다
```java
studentStream.sorted(Comparator.comparing(Student::getBan)
                                .thenComparing(Student::getTotalScore)
                                .thenComparing(Student::getName))
                                .forEach(System.out::println);              // 학생들을 반, 총점, 이름순으로 정렬
```
Comparator.naturalOrder 함수와 Comparator.reverseOrder 함수는 구현한 Comparator.compareTo 함수를 사용해 정렬한다  
다음은 naturalOrder를 사용해 정렬한 예시이다
```java
public static void main(String[]args){
    List<Student> studentList = new ArrayList<>();
    studentList.add(new Student("A", 30));
    studentList.add(new Student("B", 20));
    studentList.add(new Student("C", 10));
    studentList.add(new Student("D", 20));
    studentList.add(new Student("E", 30));

    studentList.stream().sorted(Comparator.comparing(Student::getAge)
                                          .thenComparing(Comparator.naturalOrder()))
                                .forEach(s -> System.out.println("s.name = " + ((Student)s).name));
}

static class Student implements Comparable{
    String name;
    Integer age;
    Integer score;

    public int compareTo(object o) {
      return this.name.compareTo(((Student) o).name);
    }
}
```


### 변환 - map()
스트림의 요소들을 특정 형태로 변환해야 할 때 사용한다  
Stream.map() 함수의 선언부는 다음과 같다
```java
Stream<<R> map(Function<? super T, ? extends R> mapper)
```
다음은 Stream<File>을 매개변수로 받아 Stream<String>으로 변환하는 예시이다
```java
Stream<String> stringStream = fileStream.map(File::getName);
```
map은 중간 연산이므로 이를 중첩 사용하면 다음과 같이 파일들의 확장자를 출력하는 코드를 작성할 수 있다
```java
fileStream.map(File::getName)
            .filter(s -> s.indexOf('.')!=-1)
            .map(s -> s.subString(s.indexOf('.')+1))
            .map(s -> s.toUpperCase)
            .distinct()
            .forEach(System.out::print);
```

### 조회 - peek()
연산과 연산 사이에 올바르게 처리가 되었는지 확인하기 위해서 peek()를 사용하자  
forEach와 달리 스트림 요소들을 소모하지 않으므로 연산 사이에 여러 번 끼워 넣어도 문제가 되지 않는다
```java
fileStream.map(File::getName)
            .filter(s -> s.indexOf('.')!=-1)
            .peek(s -> System.out.println(s))           // 확장자가 없는 파일 제외하고 출력
            .map(s -> s.subString(s.indexOf('.')+1))
            .peek(s -> System.out.println(s))           // 확장자만 잘라 출력
            .map(s -> s.toUpperCase)
            .peek(s -> System.out.println(s))           // 확장자를 대문자로 변환해 출력
            .distinct()
            .forEach(System.out::print);
```

### 기본형 스트림 - mapToInt, mapToLong, mapToDouble
기본적으로 스트림 연산은 Stream<T>타입의 스트림을 출력한다  
하지만 기본형 요소들을 출력하는 경우 다음과 같이 기본형 스트림을 사용하는 것이 더 편리하다  
다음과 같이 학생들의 총점을 더해 출력하는 경우를 생각해보자
```java
Stream<Integer> integerStream = studentStream.map(Student::getAge);
for (Object o : integerStream.toArray()) {
    sum += ((Integer) o).intValue();
}

```
mapToInt 함수를 사용하면 Integer를 int로 변환할 필요가 없다  
또한 IntStream과 같은 기본형 스트림은 sum()과 같은 유용한 메서드를 제공한다
```java
int sum = studentStream.mapToInt(Student::getAge).sum();
int avg = studentStream.mapToInt(Student::getAge).average();
```
이러한 메서드는 모두 최종 연산이므로 연속적인 사용이 불가능하다  
하지만 이러한 메서드는 같이 사용되는 경우가 많으므로 summaryStatistic() 라는 메서드가 제공된다
```java
IntSummaryStatistics stat = studentStream.mapToInt(Student::getAge).summaryStatistics();
long totalCount = stat.getCount();
long sum = stat.getSum();
double avgScore = stat.getAverage();
int minScore = stat.getMin();
int maxScore = stat.getMax();
```

### 타입 변환 - flatMap(), toArray()
스트림의 요소가 Stream<T> 가 아닌 Stream<T[]> 인 경우, 각 배열의 요소들에 대해 일괄적인 처리를 하기 어렵다  
만약 Stream<T[]>에 대해 map(Arrays::stream)을 사용하면, 다음과 같이 Stream<Stream<T>>를 반환한다
```java
Stream<String[]> stringArrayStream = Stream.of(["aaa","bbb"],["ccc","ddd"],["eee","fff"]);
Stream<Stream<String>> stringStreamStream = stringArrayStream.map(Arrays::stream())
```
우리가 원하는 결과는 Stream<String>이므로 이를 얻으려면 flatMap()을 사용해야 한다
```java
Stream<String[]> stringArrayStream = Stream.of(["aaa","bbb"],["ccc","ddd"],["eee","fff"]);
Stream<String> stringStreamStream = stringArrayStream.flatMap(Arrays::stream());
```
Stream<T> 타입을 다시 T[] 타입으로 변환하려면 toArray()메서드를 사용하면 된다  
toArray() 메서드의 반환 타입은 Object[]이므로 다음과 같이 생성자를 지정해 주어야 한다
```java
String[] stringArray = stringStream.toArray(String::new);
```

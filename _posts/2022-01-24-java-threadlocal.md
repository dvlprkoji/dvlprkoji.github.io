---
title: 자바 쓰레드 로컬(ThreadLocal)
date: 2022-01-24 00:00:01
categories: [Java]
tags: [java, threadlocal]     # TAG names should always be lowercase
---

## ThreadLocal?

ThreadLocal은 해당 쓰레드만 접근할 수 있는 특별한 저장소를 말한다.
쉽게 이야기해서 물건 보관 창구를 떠올리면 된다.
여러 사람이 같은 물건 보관 창구를 사용하더라도 창구 직원은 사용자를 인식해서 사용자별로 확실하게 물건을 구분해준다.
이해하기 쉽도록 예시를 들어 보겠다.


### ThreadLocal이 아닌 필드 변수를 사용할 경우

1. threadA가 필드 변수에 userA라는 값을 저장한다

![image](https://user-images.githubusercontent.com/46219687/150712854-2d96741c-c7a8-45a6-a2c8-4f7bc895277c.png)


2. threadB가 필드 변수에 userB라는 값을 저장한다

![image](https://user-images.githubusercontent.com/46219687/150713005-7d6d1534-a334-460d-b1f8-7791b9552f68.png)

이후 threadA가 필드 변수에 저장된 값을 불러온다면 userA가 아닌 userB값이 불러질 것이다


### ThreadLocal을 사용할 경우

1. threadA가 쓰레드 로컬에 userA라는 값을 저장한다

![image](https://user-images.githubusercontent.com/46219687/150713360-f9bf9cdf-bb4f-458f-aa1e-21ccab581693.png)

2. threadB가 쓰레드 로컬에 userB라는 값을 저장한다

![image](https://user-images.githubusercontent.com/46219687/150713412-bf03e176-125c-48bd-b3c5-8b1a8cf97de4.png)

3. 이후 쓰레드 로컬을 사용하는 쓰레드에 따라 저장된 값을 다르게 불러온다

![image](https://user-images.githubusercontent.com/46219687/150713513-2c8ae514-46cf-44c9-9fc9-cf143c6577c3.png)


## ThreadLocal 사용 주의사항

ThreadLocal에 저장된 값을 사용 후 제거하지 않고 그냥 두면 WAS처럼 쓰레드 풀을 사용하는 경우 심각한 문제가 발생할 수 있다.
다음 예시를 통해 알아보자.

### 1. 사용자A 저장 요청

![image](https://user-images.githubusercontent.com/46219687/150714581-da8e900d-6ff9-430c-b4d3-2cd978d8a042.png)
1. 사용자A가 저장을 요청한다
2. 쓰레드 풀에서 쓰레드를 하나 조회한다
3. threadA를 할당한다
4. threadA는 쓰레드 로컬에 사용자A를 저장한다
5. 쓰레드 로컬은 threadA 전용 보관소에 사용자A를 저장한다

### 2. 사용자A 저장 요청 종료

![image](https://user-images.githubusercontent.com/46219687/150718620-3d79b132-08b6-4c06-91de-1e6fea76b672.png)
1. WAS에서 사용자에게 요청에 대한 응답
2. threadA 반환
(아직 쓰레드 로컬에 사용자A 정보가 남아있음)


### 3. 사용자B 조회 요청

![image](https://user-images.githubusercontent.com/46219687/150719195-f1161572-9c98-4f04-8c50-660707fae4a5.png)
1. 사용자B가 조회를 요청한다
2. 쓰레드 풀에서 쓰레드를 하나 조회한다
3. threadA를 할당한다
4. threadA는 쓰레드 로컬에 저장된 사용자를 조회한다
5. 쓰레드 로컬은 threadA 전용 보관소에서 사용자A를 응답한다
6. threadA는 사용자A를 전달받는다
7. 사용자B에게 사용자A를 응답한다


### 결론

결과적으로 사용자B는 사용자A의 데이터를 확인하게 되는 심각한 문제가 발생하게 된다.
이런 문제를 예방하려면 사용자A의 요청이 끝날 때 쓰레드 로컬의 값을 ```ThreadLocal.remove()```를 통해 꼭 제거해야 한다.


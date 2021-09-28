---
title: 스프링 의존관계 주입 (Dependency Injection)
date: 2021-09-28 00:00:01
categories: [Spring, Core]
tags: [spring, dependency injection]     # TAG names should always be lowercase
---

#의존관계 주입이란?
어플리케이션이 실행된 후 사용할 객체들이 생성된다  
생성된 객체들이 서로를 의존하는 경우 각 객체들의 이름을 모를 뿐더러 주입하는 코드를 작성하기엔 너무 불편하다    
따라서 생성된 객체들의 의존관계를 자동으로 설정할 필요가 있는데 이를 구현하는 다양한 방법이 존재한다  
하나씩 알아보고 어떤 주입을 사용해야 하는지, 더 편하게 사용하는 방법은 없는지 알아보자

###1. 생성자 주입
```java
public class MemberController{
    private final MemberService memberService;
    
    @Autowired
    public MemberController(MemberService memberService){       // 생성자 주입
        this.memberService = memberService;
    }
}
```
이름 그대로 생성자를 통해 의존 관계를 주입받는 방법이다  
생성자는 객체 생성시 한번만 호출되므로 유일성이 보장되므로 **유일성을 보장**한다는 특징이 있다  
위와 같이 생성자가 하나만 존재할 경우 ```@Autowired``` 어노테이션을 생략할 수 있다
###2. setter 주입
```java
public class MemberController{
    private final MemberService memberService;

    @Autowired
    public setMemberService(MemberService memberService){       // setter 주입 (수정자 주입)
        this.memberService = memberService;
    }
}
```
자바빈 프로퍼티의 수정자 메서드 방식을 사용하는 방식이다  
setter는 생성 이후에도 사용할 수 있으므로 **선택, 변경 가능성이 있는** 의존관계에 사용한다
###3. 필드 주입
```java
public class MemberController{
    @Autowired
    private final MemberService memberService;                  // 필드 주입
}
```
말 그대로 필드에 주입하는 방식이다  
코드에 직접 주입하기에 매우 간편하지만 외부에서 변경이 불가능해서 테스트가 불가능하다는 단점이 있다
###4. 메서드 주입
```java
public class MemberController{
    private final MemberService memberService;
    pirvate final LoginService loginService;
    @Autowired
    public init(MemberService memberService, LoginService loginService){       // 메서드 주입
        this.memberService = memberService;
        this.loginService = loginService;
    }
}
```
여러 멤버를 한번에 주입받을 수 있지만 잘 사용하지 않는다

# 의존관계 주입 사용 가이드
> ###생성자 주입을 선택해라!
과거에는 setter 주입과 필드 주입을 많이 사용했지만 최근에는 대부분의 프레임워크가 생성자 주입을 권장한다    
그 이유는 다음과 같다  

####불변
대부분의 의존관계는 바뀌지 않는다
####접근 제어자
setter 주입을 사용하려면 setter메서드의 접근 제어자를 public으로 열어 두어야 한다  
setter의 접근을 막아야 하는 경우에도 public 설정이 강제된다 
####누락
생성자 주입을 사용하면 주입 데이터를 누락했을 때 컴파일 오류가 발생한다  
컴파일 시점에 오류를 발견할 수 있다는 장점이 있다

# Lombok을 사용하는 의존관계 주입
이제 의존관계 주입에 사용될 방식을 결정했다  
하지만 필드 주입을 사용하다가 생성자 주입을 사용하려니 여간 불편한 것이 아니다  
이러한 불편함을 해소하기 위해 **Lombok**이라는 라이브러리가 존재한다
```java
@RequiredArgsConstructor
public class MemberController{
    private final MemberService memberService;
    pirvate final LoginService loginService;
}
```
Lombok의 ```@RequiredArgsConstructor``` 어노테이션은 private이 지정된 필드를 모아 생성자를 만들어 준다  
결과적으로 필드 주입과 같은 형태로 생성자 주입을 사용하는 것이다

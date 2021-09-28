---
title: 스프링 컨테이너(Container)와 스프링 빈(Bean)
date: 2021-09-278 00:00:01
categories: [Spring, Core]
tags: [spring, container, bean]     # TAG names should always be lowercase
---
##스프링 컨테이너
**스프링 컨테이너**는 스프링 빈을 관리하는 객체이다  
스프링 컨테이너는 ```@Configuration```이 붙은 설정 클래스를 설정 정보로 사용한다  
이 클래스 내부에서 ```@Bean```이 붙은 메서드의 이름으로 컨테이너에 객체를 등록하는데 이를 **스프링 빈**이라고 한다

##스프링 빈
**스프링 빈**은 프로그램 실행 중 사용하기 위해 미리 만들어놓은 객체라고 생각하면 된다
<br><br>

##스프링 빈의 생명주기
####1. 스프링 컨테이너 생성
스프링 컨테이너를 생성할 때에는 구성 정보를 지정해 주어야 한다  
구성 정보는 class파일, XML파일, Groovy파일 등등 다양한 형식이 지원된다  
이번 포스팅에선 class파일을 사용한 구성을 사용한다
####2. 스프링 빈 등록
스프링 부트의 설정파일이 다음과 같다고 하자
```java
// AppConfig.class
@Configuration
public class AppConfig{
    @Bean
    public MemberController(){return new MemberController(memberService);}
    @Bean
    public MemberService(){return new MemberServiceImpl(memberRepository);}
    @Bean
    public MemberRepository(){return new MemberRepository();}
}
```

```@SpringBootApplication```은 위```AppConfig.class```설정파일을 넘겨받아 스프링 컨테이너를 생성한다

```java
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
```
이후 ```@Bean```이 붙은 메서드의 이름으로 스프링 빈을 생성하고 스프링 컨테이너에 등록한다

####3. 의존관계 주입
이후 등록된 빈에 의존관계를 주입하는데 ```@Configuration```어노테이션을 통해 싱글톤 패턴을 유지하면서 의존성을 주입한다  

####4. 초기화 콜백
초기화 콜백을 지원하는 다양한 방법들이 있지만, 최신 스프링에서 권장하는 방법은 ```@PostConstruct```를 사용하는 것이다
####5. 사용
####6. 소멸전 콜백
초기화 콜백과 동일하게 ```@PreDestroy``` 어노테이션을 지원한다
####7. 스프링 종료

---
title: 서블릿 컨테이너와 스프링 컨테이너
date: 2021-09-27 00:00:01
categories: [Spring, Core]
tags: [spring, servlet container, spring container]     # TAG names should always be lowercase
---
# 서블릿 컨테이너란?
서블릿은 개발자로 하여금 핵심 비즈니스 로직만 처리할 수 있도록 다양한 기능들을 수행한다  
이러한 서블릿을 관리하는 서블릿들의 생성, 실행, 파괴, 즉 **서블릿의 생명주기를 관리**한다    
![image](https://user-images.githubusercontent.com/46219687/135403991-4e400931-4733-46d8-8a28-d9032d3fb724.png){: width="50%" height="50%"}
<br><br>

# 스프링 컨테이너란?
스프링 컨테이너는 **스프링 빈의 생명주기를 관리**한다  
스프링 빈의 생명주기는 다음과 같다  
```
스프링 컨테이너 생성 -> 스프링 빈 생성 -> 의존관계 주입 -> 초기화 콜백 -> 사용 -> 소멸자 콜백 -> 스프링 종료
```

#웹 어플리케이션 동작 원리
![image](https://i.imgur.com/PlDF42i.png){: width="50%" height="50%"}
1. 웹 어플리케이션이 실행되면 WAS가 web.xml을 로딩한다
2. web.xml에 등록된 ContextLoaderListener(Java Class)가 생성된다. (ContextLoaderListner는 ApplicationContext를 생성하는 역할을 한다)
3. 생성된 ContextLoaderListener는 root.xml을 로딩힌다
4. root.xml에 등록된 Spring Container를 생성하고 Spring Bean을 등록, 의존관계를 주입한다
5. 클라이언트로부터 request를 받는다
6. DispatcherServlet을 생성한다
7. DispatcherServlet은 servlet-context.xml를 로딩한다
8. 두번째 Spring Container가 구동되며 응답에 맞는 PageController들이 동작한다
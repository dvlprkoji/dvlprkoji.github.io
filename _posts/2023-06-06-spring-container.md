---
title: 스프링 컨테이너와 서블릿 컨테이너
date: 2023-06-06 00:00:01
categories: [Spring]
tags: [spring, spring container, servlet container]     # TAG names should always be lowercase
---

웹 개발자라면 스프링 컨테이너와 서블릿 컨테이너에 대해 어느 정도는 알고 있을 것이다.<br>
나도 딱 어느정도만 알고 있고, 실무에서 직접적으로 다루지는 않다 보니 공부를 해도 금세 휘발되어 버린다.<br>
웹개발자라면 컨테이너, 스프링 컨테이너, 서블릿 컨테이너에 대해 언제나 설명할 수 있어야 한다고 생각하기에 정리해본다.<br><br>

## 서블릿이란?

> 서블릿(Servlet)이란 동적 웹 페이지를 만들 때 사용되는 자바 기반의 웹 애플리케이션 프로그래밍 기술이다.

즉 정적인 웹사이트를 사용자나 요청에 따라 동적인 웹사이트로 바꿀 수 있게 도와주는 기술이다.<br>
쉽게 예를 들면 사용자의 로그인 요청을 처리하여 사용자마다 다른 웹 페이지를 보여주는 것이 바로 서블릿이다.<br><br>

## 서블릿 컨테이너란?

> 서블릿의 생성, 실행, 소멸 등 서블릿의 생명주기를 관리하는 컨테이너이다.

웹서버와 서블릿이 쉽게 통신할 수 있도록 소켓을 만들고 listen, accept 등의 API 를 제공하여 **웹서버와의 통신을 지원** 한다.<br>
또한 새로운 요청이 올 때마다 새로운 자바 쓰레드를 생성하고 Http Service() 실행 후 자동 소멸하여 **멀티스레드를 관리** 한다.<br><br>

## 스프링 컨테이너란?

> 스프링 빈의 생성, 실행, 소멸 등 스프링 빈의 생명주기를 관리하는 컨테이너다.

스프링 빈의 생명주기를 관리하는 것 뿐만 아니라 의존관계를 주입(Dependency Injection)하는 아주 중요한 일도 수행한다.<br><br>

## 웹 어플리케이션의 컨테이너 구조 및 동작 과정

스프링 웹 기술은 MVC 아키텍쳐를 근간으로 하고 있다.<br>
MVC 아키텍쳐의 경우 제일 앞에 하나의 서블릿 컨테이너를 두고 그 뒤에 스프링 컨테이너를 두는 다음과 같은 구조를 가진다.

![servlet_container](https://github.com/dvlprkoji/springbook/assets/46219687/a4b48f3a-5ec4-4c1d-a097-9b717775ba31)

<br>그렇다면 이러한 구조에서 톰캣과 같은 WAS에 웹 어플리케이션이 올라가 구동되는 과정을 살펴보자.

![spring_container](https://github.com/dvlprkoji/springbook/assets/46219687/8c5f77b2-a50f-40b1-a492-a52fca3acc42)


1. 웹 어플리케이션이 실행되면 톰캣은 DispatcherServlet 설정이 담긴 web.xml 설정을 load 한다
2. web.xml에 등록되어 있는 ContextLoaderListener를 Servlet Container에 생성한다
3. ContextLoaderListener는 MVC의 Model 설정이 담긴 root-context.xml 설정을 load 한다
4. root-context.xml에 등록되어 있는 스프링 빈들을 Spring Container(Root)에 생성한다
5. 사용자의 요청이 들어온다
6. web.xml에 등록되어 있는 DispatcherServlet을 Servlet Container에 생성한다.
7. DispatcherServlet은 MVC의 View 설정이 담긴 servlet-context.xml 설정을 load 한다
8. servlet-context.xml에 등록되어 있는 컨트롤러들을 두번째 Spring Container에 생성한다.

이후 생성된 MVC 아키텍쳐이 서로 협업하여 요청을 처리하게 되고 이 과정은 다음 포스팅에 작성해보도록 하겠다.

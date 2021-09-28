---
title: Spring Security 개요
date: 2021-09-28 00:00:01
categories: [Spring, Security]
tags: [spring security]     # TAG names should always be lowercase
---
 # Spring Security란?
필자는 이전에 필터를 사용해 로그인 처리를 직접 구현한 경험이 있다.  
그리 복잡하지 않았던 프로젝트임에도 불구하고 페이지가 추가될때마다 추가적인 설정을 해주는 것이 매우 불편했다  
실제 서비스에서 이렇게 귀찮은 일을 할리가 없다고 생각하여 이러한 기능을 구현한 라이브러리가 있는지 찾아보았다  
아니나다를까 Spring 프로젝트의 하위 프로젝트로 Spring Security라는 것이 있었다  
그렇다면 Spring Security란 어떤 프로젝트일까?  
위키에 정의된 Spring Security는 다음과 같다  
>Spring Security는 엔터프라이즈 애플리케이션에 대한 **인증**, **권한 부여** 및 **기타 보안 기능**을 제공하는 Java / Java EE 프레임 워크입니다.

정리를 해보자면 다음 3가지 기능을 제공한다는 것이다
1. 인증 (Authentication)
2. 인가 (Authorization)
3. 기타 보안 기능 (ex. OAuth 2.0)

그렇다면 Spring Security는 어떤 방식으로 이를 구현했을까?  
Spring Security는 기본적으로 필터를 통해 이를 구현한다  
스프링 필터에 대한 내용은 이전 포스팅을 참고하자

<br><br>
![Filter Chain](https://slack-imgs.com/?c=1&url=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F226F4C3A589AA1871D)
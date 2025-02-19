---
title: 스프링 MVC
date: 2023-06-07 00:00:01
categories: [Spring]
tags: [spring, spring mvc]     # TAG names should always be lowercase
---

저번 포스팅에서 서블릿 컨테이너, 스프링 컨테이너의 생성 과정을 알아 보았다.<br>
이번 포스팅에서는 MVC 아키텍쳐에서 각 컨테이너들의 생성 이후 사용자의 요청을 처리하는 과정을 알아 본다.

## MVC 아키텍쳐란?

> 프레젠테이션 계층의 구성요소를 **정보를 담은 모델(M)**, **화면 출력 로직을 담은 뷰(V)**, **제어 로직을 담은 컨트롤러(C)** 로 분리하고
이 세 가지 요소가 서로 협력하여 하나의 웹 요청을 처리하고 응답을 만들어내는 구조다.

일반적으로 MVC 아키텍쳐는 **프론트 컨트롤러 패턴**과 함께 사용된다.<br>
프론트 컨트롤러 패턴은 중앙집중형 컨트롤러를 프레젠테이션 계층의 제일 앞에 둬서 서버로 들어오는 모든 요청을 먼저 받아 처리한다.<br><br>

## DispatcherServlet과 MVC의 동작 과정

MVC의 각 요소와 프론트 컨트롤러가 어떻게 협력하여 일하는지 파악하고 있어야 복잡한 웹 프레젠테이션 계층의 로직을 구현할 수 있다.<br>
서버가 브라우저나 여타 HTTP 클라이언트로부터 HTTP 요청을 받기 시작해서 다시 HTTP로 결과를 응답해주기까지의 과정을 살펴보자.<br>

![스크린샷 2023-06-07 195950](https://github.com/dvlprkoji/springbook/assets/46219687/cc118182-8596-44d0-b95d-454f5c76723f)

### 1. DispatcherServlet의 HTTP 요청 접수
저번 시간에 알아본 서블릿 컨테이너는 HTTP 프로토콜을 통해 들어오는 요청이 DispatcherServlet에 할당된 것이라면
HTTP 요청정보를 DispatcherServlet에 전달해준다.
DispatcherServlet은 web.xml에 작성된 서블릿-매핑 정보를 기반으로 자신이 전달받을 URL 패턴이라면
등록된 전처리 작업(보안, 파라미터 조작, 한글 디코딩 등)을 수행한다.<br>

### 2. 컨트롤러로 HTTP 요청 위임
DispatcherSerlet은 HTTP 요청 정보(URL, 파라미터 등)를 참고해 어떤 컨트롤러에 작업을 위임할지 결정한다.
컨트롤러를 선정하는 것은 DispatcherServlet의 **핸들러 매핑 전략**을 이용한다.
어떤 컨트롤러가 요청을 처리하게 할지를 결정했다면, 다음은 해당 컨트롤러의 메소드를 호출해 웹 요청을 처리하도록 해야한다.
그렇다면 DispatcherServlet은 호출할 각 컨트롤러의 정보를 다 알고 있어야 할까?
그렇지 않다. 컨트롤러를 호출할 때에는 다음과 같이 해당 컨트롤러의 타입을 지원하는 어댑터를 중간에 껴서 호출한다.<br>

![image](https://github.com/dvlprkoji/springbook/assets/46219687/1ab0470a-8fa0-440f-af23-11d708fc7725)<br>

DispatcherServlet이 핸들러 어댑터에 웹 요청을 전달할 때는 요청 정보가 담긴 HttpServletRequest 오브젝트를 전달한다.
이를 어댑터가 적절히 변환하여 컨트롤러 메소드가 받을 수 있는 파라미터로 전달해주는 것이다.<br>

### 3. 컨트롤러의 모델 생성
컨트롤러는 사용자 요청을 해석하고 실제 비즈니스 로직을 수행할 서비스 계층 오브젝트에 작업을 위임한다.
그리고 모델을 생성해 서비스 계층 오브젝트의 작업 결과를 넣어준다.<br>

### 4. 컨트롤러의 모델, 뷰 리턴
모델을 생성하고 나서, 뷰를 결정해야 한다. 컨트롤러가 뷰를 직접 전달할수도 있지만, 보통은 뷰의 논리적인 이름을 리턴해주면
DispatcherServlet의 전략인 뷰 리졸버가 이를 이용해 뷰를 생성해준다.
모델과 뷰를 생성해 DispatcherServlet에 넘겨주고 나면, 컨트롤러의 책임은 끝이다.<br>

### 5. 뷰에 모델 전달
DispatcherServlet은 뷰 오브젝트에 모델을 전달하고 클라이언트에 돌려줄 최종 결과물을 생성해달라고 요청한다.

### 6. 결과물 생성 및 전달
뷰는 전달된 모델의 정보를 참고하여 결과물을 동적으로 생성한다. 결과물은 HTML 이외에도 엑셀, PDF 등이 될 수 있다.
생성된 결과물은 HttpServletResponse에 담겨 DispatcherServlet에 전달된다.

### 7. DispatcherServlet의 HTTP 응답 전달
뷰 생성까지의 모든 작업을 마쳤으면 DispatcherServlet은 등록된 후처리기가 있는지 확인하고,
후처리기가 있다면 처리를 진행한 후에 뷰가 만들어준 HttpServletResponse를 서블릿 컨테이너에 전달한다.
서블릿 컨테이너는 HttpServlet에 담긴 정보를 HTTP 응답으로 만들어 클라이언트에 전달하고 작업을 종료한다.

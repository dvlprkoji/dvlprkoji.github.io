---
title: java stream
date: 2021-09-25 14:29:30 -4000
categories: [Java, Basic]
tags: [java, stream]     # TAG names should always be lowercase
---


# 카운셀러몰

아모레퍼시픽 카운셀러몰의 퀴즈, 컨텐츠메뉴 admin page web application 입니다.

# 사용 버전

- JAVA 11
- Spring Boot 2.5.2

# 실행 스크립트

```bash
#!/bin/bash

if [ -f process.pid ]; then
    kill -9 `cat process.pid`
fi

nohup java -jar process.jar > /dev/null 2>&1 & echo $! > process.pid
```

# 기술 스택

## Server-side

- 웹 프레임워크 : [Spring](http://https://spring.io/)
- ORM : [Mybatis](https://mybatis.org/mybatis-3/)
- 데이터베이스 : [PostgreSQL](https://www.postgresql.org/)

## Client-side

- 뷰 레이어 : [Javascript](https://facebook.github.io/react/), [AJAX](https://api.jquery.com/jquery.ajax/)
- 프론트엔드 프레임워크 : [Bootstrap](getbootstrap.com)

## Tools

- 프로젝트 형상관리 : [Gitlab](https://about.gitlab.com/)
- CI/CD : [Jenkins](https://www.jenkins.io/)
- WAS : [Apache Tomcat](http://tomcat.apache.org/)
- 이미지 서버 : [AWS S3](https://aws.amazon.com/ko/s3/)

# 폴더 구조

```bash
├── README.md                 - 리드미 파일
│
├── src/                      - 어플리케이션 폴더
│   └── main                  - 서버 사이드 소스 코드
│       ├── java/             - 메인 패키지
│       ├── resources/        - properties 정의
│       └── webapp/           - 클라이언트 사이드 소스 코드
│
├── out/artifacts             - build 결과물
├── META-INF                  - application XML 정의
├── .mvn/wrapper              - maven 설정
├── mvnw                      - maven 설정
├── mvnw.cmd                  - maven 설정
├── pom.xml                   - dependency 설정
└── .gitignore                - gitignore 파일
```

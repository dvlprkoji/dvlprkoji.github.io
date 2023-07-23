---
title: 스프링 배치(Spring Batch) - 1. 스프링 배치 시작
date: 2023-07-23 00:00:01
categories: [Spring]
tags: [spring, batch]     # TAG names should always be lowercase
---

최근 업무 중 스프링 배치를 사용할 일이 있어 빠르게 사용법을 익히고 적용하게 되었다.<br>
기존에 인프런에서 강의를 사놓은 게 있어 참고하긴 했지만 너무 빠르게 익힌 감이 있어 글로 정리해 보려 한다.<br>

## 배치 애플리케이션이란?

배치(Batch)란 ***일괄처리*** 란 뜻을 가지고 있다.<br>
만약 매일 전 날 데이터를 집계 해야한다고 가정해 보자.<br>
이 집계 과정을 기존의 웹 어플리케이션에서 수행한다면 커다란 문제가 생길 것이다.<br>
집계하는 과정에서 아주 커다란 데이터를 읽고, 가공하고, 저장하기 때문에 서버의 CPU, I/O 등의 자원을 다 써버려서<br>
[기존의 웹 어플리케이션에서 수행하던 Request 처리를 하지 못하게 될 것이다.<br><br>

그리고 이 집계 기능은 하루에 1번 수행된다.<br>
이를 위해 복잡한 API를 구성하는 것은 낭비라고 볼 수 있다.<br>
또한 성격이 다른 집계 기능을 추가할 때에도 비즈니스 로직 개발이 아닌 읽기, 가공하기, 저장하기 등의 기능을 <br>
개발하느라 대부분의 시간을 쓸 것이다. <br><br>

그리고 만약 이 집계 기능을 수행하는 도중 실패했을 때 이를 처리하는 과정도 매우 복잡할 것이다. <br>
5만번째에서 실패했을때 이를 skip 하고 5만1번째 데이터를 처리하도록 하고 싶어도 어려움이 있을 것이다. <br>
또 이런 경우도 있을 수 있다. 오늘 아침 누군가가 집계 함수를 실행시켰는데, <br>
다른 누군가가 또 실행시켜 집계 데이터가 2배로 뻥튀기 될 수도 있다. <br>
같은 파라미터로 같은 함수를 실행할 경우 실패하도록 할 수 있다면 이를 방지할 수 있을 것이다 <br><br>

바로 이렇게 단발성으로 대용량의 데이터를 처리하는 어플리케이션을 배치 어플리케이션이라고 한다. <br>
위 고민들의 공통점을 생각해보면 결국 비즈니스 로직이 아닌 배치성 기능을 처리해줄 무언가가 필요하다는 결론이 나온다.<br>
Spring 진영에서 이런 배치 어플리케이션을 지원하는 모듈로 Spring Batch가 있다. <br><br>


### 스프링 배치 시작 - @EnableBatchProcessing

스프링 배치를 시작하기 위해 의존성 설정을 해준 뒤, 가장 먼저 해야 할 것은 ```@EnableBatchProcessing``` 어노테이션 지정이다 <br>
해당 어노테이션을 지정하면 총 4개의 설정 클래스를 실행시키고 배치의 모든 초기화 및 실행 구성을 한다. <br>
다음은 ```@EnableBatchProcessing``` 어노테이션이 실행하는 3개의 설정 클래스이다.<br>

1. BatchAutoConfiguration
   - Job을 실행하는 JobLauncherApplicationRunner 빈을 생성
2. SimpleBatchConfiguration
   - JobBuilderFactory, StepBuilderFactory 등 스프링 배치의 주요 구성 요소를 프록시 객체로 생성
3. BatchConfigurerConfiguration
   - SimpleBatchConfiguration 에서 생성한 프록시 객체를 기반으로 실제 대상 객체를 생성

한 문장으로 ```@EnableBatchProcessing``` 어노테이션의 기능을 설명하자면 <br>
***스프링 배치의 주요 구성 요소를 생성하고 빈으로 등록한 뒤 JobLauncherApplicationRunner를 실행시켜 빈으로 등록된 Job을 실행***<br>
한다고 할 수 있다.<br><br>


### 스프링 배치 시작 - DB 스키마 생성

스프링 배치를 위해 어노테이션을 지정한 후 또 해야할 일이 있다. <br>
스프링 배치는 관리를 위한 목적으로 여러 도메인들 (Job, Step, JobParameter 등)의 정보들을 관리할 수 있는 스키마를 제공한다.<br>
RDBMS를 사용할 수도 있고 주로 테스트 목적으로 사용하는 인메모리 DB(H2 등)를 사용할 수도 있다.<br>
얼마 전 업무에서는 Spring Cloud Data Flow (클라우드 기능) 내부 DB에 연결에 사용하였다. <br>
아무튼 스프링 배치 메타데이터를 저장할 DB 스키마를 생성했다면 Datasource 설정을 해야 한다. <br><br>

일반적으로 Spring Batch를 사용하는 어플리케이션에서는 배치 메타데이터를 저장하는 DB 뿐만 아니라 비즈니스 로직에 사용되는 DB도 사용한다. <br>
그렇다면 다중 DataSource 설정을 해야 할텐데 이는 배치 메타데이터 관리용 DB 설정에 ```@Primary``` 어노테이션을 붙이면 된다

```java

@Configuaration
public class DataConfig {

    @Primary
    @Bean(name = "batchDataSource")
    @ConfiguartionProperties(prefix = "spring.datasource.batch")
    public DataSource batchDataSource() {
        return DataSourceBuilder.create.build();
    }

    @Bean(name = "oracleDataSource")
    @ConfiguartionProperties(prefix = "spring.datasource.oracle")
    public DataSource oracleDataSource() {
      return DataSourceBuilder.create.build();
  }
}
```

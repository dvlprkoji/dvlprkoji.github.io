---
title: 스프링 배치(Spring Batch) - 2. 스프링 배치 도메인 이해
date: 2023-07-27 00:00:01
categories: [Spring]
tags: [spring, batch]     # TAG names should always be lowercase
---

저번 포스팅에서 스프링 배치를 구성하기 위해 필요한 준비들에 대해 알아보았다. <br>
이번 포스팅에서는 스프링 배치에서 사용하는 주요 도메인들에 대해 알아볼 것이다. <br>

### Job

배치 계층 주소에서 가장 상위에 있는 개념으로 하나의 배치작업 자체를 의미한다. <br>

### JobParameter

Job을 실행할 때 함께 포함되어 사용되는 파라미터를 가진 객체이다. <br>

### JobInstance

Job 이 실행될 때 생성되는 Job 의 논리적 실행 단위 객체이다. <br>
만약 처음 실행하는 Job + JobParameter 일 경우 새로운 JobInstance 를 생성한다. <br>
이미 실행한 적 있는 Job + JobParameter 일 경우 기존에 생성된 JobInstance를 리턴한다 <br>

다음은 "일별 정산"을 하는 Job 에 대해 "2021.01.01" 이라는 JobParameter 를 전달하여 실행한 경우를 도식화한 것이다. <br><br>
![image](https://github.com/dvlprkoji/dvlprkoji/assets/46219687/f683b38c-5c6f-45a8-99b8-dba04d1067be)

다음날 동일 Job에 대해 "2022.01.02" 라는 JobParameter 로 실행하면 다음과 같이 BATCH_JOB_INSTANCE 테이블에 저장될 것이다. <br><br>
![image](https://github.com/dvlprkoji/dvlprkoji/assets/46219687/87888a0e-e93a-4294-8478-48e4030be9fe)


### JobExecution

JobInstance 에 대한 한번의 시도를 의미하는 객체이다. <br>
시작시간, 종료시간, 상태, 종료상태의 속성을 가진다. <br>
JobExecution 의 실행 상태가 ***COMPLETED*** 가 될 때까지 하나의 JobInstance 내에서 여러 번의 시도가 생길 수 있다.<br>

다음은 위 예제에서 살펴본 일별 정산 Job 에 대해 어떻게 JobExecution 이 생성되는지 도식화한 것이다. <br><br>

![image](https://github.com/dvlprkoji/dvlprkoji/assets/46219687/95b259b5-8317-4244-8bc4-789297bc22f7)


### Step

Job을 구성하는 독립적인 하나의 단계이다. <br>
Step의 종류로 JobStep, TaskletStep, FlowStep, PartitionStep 이 있다. <br>


### StepExecution

Step 에 대한 한번의 시도를 의미하는 객체이다. <br>
시작시간, 종료시간, 상태 등의 속성을 가진다. <br>
Step 이 실패할 경우 StepExecution 은 FAILED 상태가 되고 JobExecution 또한 FAILED 상태가 된다. <br>
다음은 이러한 상황을 잘 보여준다. <br><br>

![image](https://github.com/dvlprkoji/dvlprkoji/assets/46219687/151df115-5e49-4ef1-bfe2-c741ebfdb21a)


### JobRepository

배치 작업 중 발생하는 정보를 저장하는 저장소 객체이다. <br>
배치 작업의 수행과 관련된 모든 메타데이터를 저장한다. <br>

### JobLauncher

Job 을 실행시키는 역할을 한다. <br>
Job 과 JobParameter 을 인자로 받아 요청된 배치 작업을 수행한 후 최종 client 에게 JobExecution 을 반환한다. <br>
Job 을 실행하는 방식으로 ***동기적 실행***과 ***비동기적 실행***이 있다. <br>
비동기적 실행의 경우 실행 즉시 응답값을 반환하는 차이점이 있다. <br><br>

![image](https://github.com/dvlprkoji/dvlprkoji/assets/46219687/812f2c6c-5d01-4511-a333-d4b770799b7a)


지금까지 알아본 배치 도메인 말고도 다양한 도메인들이 있지만 주요 도메인들만 알아 보았다.<br>
다음 포스팅에서는 실제 실행 과정에 대해 알아 보자. <br>

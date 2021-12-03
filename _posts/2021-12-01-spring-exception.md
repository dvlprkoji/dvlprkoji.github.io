---
title: 스프링 예외(Exception)
date: 2021-12-01 00:00:01
categories: [Spring, Core]
tags: [spring, exception]     # TAG names should always be lowercase
---

잘못된 예외 처리 코드는 찾기 힘든 버그를 낳을 수 있다
대표적인 잘못된 예외 처리 코드를 알아보고 예외 처리 전략도 알아보자

### 예외 블랙홀

```java
try{
    // SQL 쿼리 전송
} catch(SQLException e){
}
```
예외가 발생하면 그것을 잡아내는 것은 좋은데, 아무것도 아닌 것처럼 넘어가면 문제가 발생한다
결국 발생 예외로 인해 어떤 기능이 비정상 동작하거나, 메모리나 리소스가 소진되는 경우가 발생할 것이다

비슷한 코드로 다음과 같은 코드들이 있다

```java
try{
    // SQL 쿼리 전송
} catch(SQLException e){
    System.out.println(e.getMessage());
}
```
발생한 예외를 콘솔에 출력해 주고는 있지만, 이는 그냥 넘어갈 확률이 높기 때문에 동일하다고 볼 수 있다
그렇다면 어떻게 처리해야 할까

예외를 처리할 때 반드시 지켜야 할 핵심 원칙은 한 가지다
모든 예외는 적절하게 복구되던지 작업을 중단하고 관리자에게 분명히 통보되어야 한다

### 무책임한 throws

catch 블록으로 예외를 잡아 봐야 해결할 방법도 없고,
수많은 API나 라이브러리가 던지는 예외들을 매번 throws로 처리하기 귀찮아지면 다음과 같은 코드가 탄생한다


```java
public class Class1{
  public void method1() throws Exception{
      method2();
  }
  public void method2() throws Exception{
      method3();
  }
  public void method3() throws Exception{}
}
```

이러한 메서드를 통해 의미있는 정보를 얻을 수 없다
단순히 습관적으로 붙인 것인지, 어떤 예외가 발생할 수 있는 것인지 알 수 없게 되어버린다
또한 적절히 처리될 수 있는 예외들도 그냥 넘어가는 문제가 발생할 수 있다


## 예외의 종류와 특징
자바에서 throw를 통해 발생시킬 수 있는 예외에는 3가지가 있다
### Error
첫째는 java.lang.Error 클래스의 서브클래스들이다
에러는 시스템적 문제가 발생하였을 때 사용된다
예를 들면 OutOfMemoryError나 ThreadDeath같은 것이 있다
이러한 에러는 처리할 수 있는 것이 없으므로 신경쓰지 않아도 된다

### Exception과 체크 예외
java.lang.Exception 클래스와 그 서브클래스로 정의되는 예외들은
개발자들이 어플리케이션 코드의 작업 중 예외상황이 발생했을경우 사용된다

Exception 예외는 **체크 예외**와 **언체크 예외**로 나뉘는데
언체크 예외는 Exception.RuntimeException 클래스를 상속받은 예외를 말하고
체크 예외는 상속받지 않은 예외를 말한다

체크 예외가 발생할 수 있는 메소드를 사용할 경우 반드시 예외를 처리하는 코드를 함께 작성해야 한다
언체크 예외는 런타임 클래스를 상속받기에 런타임 예외라고 불리기도 한다

먼저 예외를 처리하는 일반적인 방법을 생각해보고 효과적인 예외처리 전략을 생각해보자

### 예외 복구

첫 번째 예외 처리 방법은 예외상황을 파악하고 문제를 해결해서 정상 상태로 돌려놓는 것이다
예외처리를 강제하는 체크 예외는 개발자로 하여금 예외상황이 발생할 수 있음을 인식하도록 도와주고,
이에 대한 적절한 처리를 시도해보도록 요구하는 것이다

예를 들어 DB 서버에 접속하는 코드가 있다고 하면, DB서버 접속에 실패했을 경우 재시도를 해볼 수 있을 것이다

### 예외처리 회피
```java
public void add() throws SQLException{
  try{
      // JDBC API
  } catch(SQLException e){
      // 에러 출력
      throw e;
  }
}
```
이처럼 만약 예외 처리가 자신의 역할이 아니라고 생각하면 예외를 밖으로 던져버릴 수 있다
이러한 방법은 꼭 분명한 의도를 가지고 있어야 한다
던진 예외를 받은 쪽에서도 자신의 역할이 아니라고 생각하면 또 예외를 밖으로 던지게 될 것이다

### 예외처리 전환
마지막으로, 예외를 처리하는 방법에 예외 전환이 있다
예외를 받아서 적절한 예외로 전환하여 밖으로 던지는데 이는 분명 예외 회피와는 다르다
예외 전환은 보통 두 가지 목적으로 사용된다

첫째, API에서 발생한 로우레벨의 예외를 상황에 적합한 의미를 가진 예외로 변경하는 것이다
예를 들어 동일한 ID를 가진 사용자를 DB에 등록하려고 하면 SQLException을 발생시킬 것이다
이 메소드를 그대로 밖으로 던져버리면 서비스 계층에서는 왜 SQLException이 발생했는지 알 수 없을 것이다
만약 SQLException을 DuplicateUserIdException같은 예외로 바꾸어 던진다면,
서비스 계층에서는 적절한 복구 작업을 시도할 수 있을 것이다

```java
public void add() throws SQLException{
  try{
      // JDBC API
  } catch(SQLException e){
      if (e.getErrorCode() == MysqlErrorNumbers.ER_DUP_ENTRY){
        throw DuplicateUserIdException(e);
      }
      else{
        throw e;
      }
  }
}
```

보통 전환하는 예외에 원래 발생한 예외를 담아 중첩 예외를 만드는 것이 좋다
그래서 SQLException을 생성자의 파라미터로 전달했다

다음으로 예외를 처리하기 쉽게 만들기 위해 포장하는 것이다
의미를 명확하게 하려고 포장하는 것이 아니라는 점에서 첫번째 방법과는 다르다

예를 들면 EJBException을 들 수 있다
EJB 컴포넌트에서 발생하는 예외는 대부분 복구 가능한 예외가 아니므로 런타임 예외인 EJBException으로 포장해 던지는 것이다
이렇게 처리할 경우 시스템 익셉션으로 파악하고 트랜잭션을 자동으로 롤백해준다

## 예외처리 전략

### 런타임 예외의 일반화 (낙관적 예외 처리)
일반적으로 체크 예외는 꼭 처리해야 할 예외, 언체크 예외는 처리할 필요가 없는 예외라고 하였다
하지만 스프링 서버 환경에서는 이런 방식을 적용하지 않는 경우가 많다

단일 사용자 환경에서 오류가 발생했을 경우, 예를 들어 엑셀 파일과 같은 경우 오류는 반드시 복구되어야 한다
하지만 다중 사용자 환경인 서버 환경에서, 작업을 중단하고 복구할 방법은 없다
따라서 예외사항을 미리 파악하고 예외가 발생하지 않도록 하거나, 예외가 발생하면 중단하고 관리자가 알도록 해야 할 것이다

따라서 위에서 사용한 add 메소드를 런타임 예외로 전환하여 사용할 것이다
중복 ID로 인해 발생한 예외는 외부에서 충분히 처리 가능한 예외이므로 이는 그대로 놔둘 것이다

```java
public class DuplicateUserIdException extends RuntimeException{
    public DuplicateUserIdException(Throwable cause) {
        super(cause);
    }
}

public void add() throws SQLException{
  try{
      // JDBC API
  } catch(SQLException e){
      if (e.getErrorCode() == MysqlErrorNumbers.ER_DUP_ENTRY){
        throw DuplicateUserIdException(e);      // 예외 전환
      }
      else{
        throw new RuntimeException(e);          // 예외 포장
      }
  }
}
```

### 애플리케이션 예외
외부의 예외 상황이 아니라 애플리케이션 자체의 로직에 의해 발생하도록 만든 예외를 애플리케이션 예외라고 한다
예를 들어 보유한 금액 이상을 출금하는 명령이 들어왔을 때 허용한 범위를 체크한 뒤 적절한 경고를 사용자에게 보내야 할 것이다

이러한 경우, 각 상황에 따라 리턴 값을 다르게 줄 수 있다
성공할 경우 0, 실패할 경우 -1 등으로 주는 것이다
하지만, 이렇게 리턴 값으로 예외상황을 체크할 경우 리턴 값을 체졔적으로 관리하지 않으면 혼란이 올 수 있다

그 다음 방법으로, 예외 상황에 대해서는 비즈니스적인 의미를 띈 예외를 던지도록 만드는 것이다
잔고 부족의 경우 InsufficientBalanceException을 던지는 것이다

```java
try{
    BigDecimal balance = account.withdraw(amount);
} catch(InsufficientBalanceException e){
    BigDecimal availFunds = e.getAvailFunds();
    // 잔고 부족 메시지 출력
}
```

이번 포스팅에서 알아본 내용들을 정리하면 다음과 같다

- 예외를 잡아서 아무런 조취를 취하지 않거나 의미없는 throws 선언을 남발하는 것은 위험하다
- 예외는 복구하거나 적절한 예외로 전환해야 한다
- 좀더 의미있는 예외로 변경하거나, 불필요한 catch/throws를 피하기 위해 런타임 예외로 포장하자
- 복구할 수 없는 예외는 런타임 예외로 전환하자
- 애플리케이션 로직을 담기 위한 예외는 체크 예외로 만든다
- SQLException의 에러 코드는 DB에 종속되기 때문에 DB에 독립적인 예외로 전환될 필요가 있다
- 스프링은 DataAccessException을 통해 DB에 독립적으로 적용 가능한 추상화된 런타임 예외를 제공한다


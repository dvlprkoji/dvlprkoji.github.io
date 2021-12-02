---
title: 예제로 알아보는 스프링 템플릿(Template), 콜백(Callback)
date: 2021-12-01 00:00:01
categories: [Spring, Core]
tags: [spring, template]     # TAG names should always be lowercase
---

이전에 작성했던 DB 커넥션 코드들 중 UserDao의 가장 단순한 메서드인 deleteAll()를 살펴보자

```java
public class UserDao{
  public void deleteAll() throws SQLException{
      Connection c = dataSource.getConnection();

      PreparedStatement ps = c.preparedStatement("delete from users");
      ps.executeUpdate();

      ps.close();
      c.close();
  }
}
```

이 메서드에서는 공유 리소스인 Connection 과 PreparedStatement 자원을 가져와 사용한다
이는 이미 만들어놓은 리소스 풀에서 가져와 사용하는 것이기 때문에 다 사용한 후 반납을 해 주어야 한다
반납이 이루어지지 않고 계속 요청이 반복되면 요청을 처리하지 못하고 결국 서비스가 중단될 것이다

따라서 우리는 이러한 예외 사항을 고려하여 코드를 수정해야 할 것이다
다음은 이를 고려해 수정한 코드이다

```java
public class UserDao{
  public void deleteAll() throws SQLException{
      Connection c = null;
      PreparedStatement ps = null;

      try{
        c = dataSource.getConnection();
        ps = c.preparedStatement("delete from users");
        ps.executeUpdate();
      }catch(SQLException e){
          throw e;
      }finally {
          if (c != null) {
              try{
                  c.close();
              } catch (SQLException e) {
              }
          }
          if (ps != null) {
              try{
                  ps.close();
              } catch(SQLException e){
              }
          }
      }
  }

    public void getCount() throws SQLException {
        Connection c = null;
        PreParedStatement ps = null;
        ResultSet rs = null;

        try{
            c = dataSource.getConnection();
            ps = c.preparedStatement("select count(*) from users");
            rs = ps.executeQuery();
            rs.next();
            return rs.getInt(1);
        } catch (SQLException e) {
            throw e;
        } finally {
            if (rs != null) {
                try{
                    rs.close();
                } catch(SQLException e){
                }
            }
            if (ps != null) {
                try{
                    ps.close();
                } catch(SQLException e){
                }
            }
            if (c != null) {
                try{
                  c.close();
                } catch(SQLException e){
                }
            }
        }
    }
}
```
당장 문제는 해결된 것처럼 보인다
하지만 코드가 너무 복잡하며 중복되는 코드들이 너무 많아 실수가 발생하기 딱 좋다
이제 이 문제를 해결해 보자

## 변하는 것과 변하지 않는 것의 분리

만약 로직에 따라 변하는 부분을 변하지 않는 나머지 코드에서 분리할 수 있다면 어떨까?
변하지 않는 부분을 재사용할 수 있지 않을까?

### 메소드 추출

먼저 생각해 볼 수 있는 것은 변하는 부분을 메소드로 추출하는 것이다
원래 변하지 않는 부분을 추출해야 하지만, 변하는 부분을 변하지 않는 부분이 감싸고 있으므로 반대로 해본 것이다

```java
public class UserDao{
    public void deleteAll() throws SQLException{
        Connection c = null;
        PreparedStatement ps = null;

        try{
            c = dataSource.getConnection();
    //      ps = c.preparedStatement("delete from users");
            ps = makeStatement(c);
            ps.executeUpdate();
        }catch(SQLException e){
            throw e;
        }finally {
            if (c != null) {
                try{
                    c.close();
                } catch (SQLException e) {
                }
            }
            if (ps != null) {
                try{
                    ps.close();
                } catch(SQLException e){
                }
            }
        }
    }

    public PreparedStatement makeStatement(Connection c) {
        return c.preparedStatement("delete from users");
    }
}
```

변하는 부분을 추출해 보았더니 당장은 별 이득이 없어 보인다
보통 분리시킨 메서드를 다른 곳에서 재사용할 수 있어야 하는데, 이건 반대로 분리시키고 남은 메서드가 재사용이 필요하다
이번엔 다른 방법으로 시도해보자

## 템플릿 메소드 패턴의 적용

템플릿 메소드 패턴이란 변하지 않는 부분을 슈퍼클래스에 두고 변하는 부분을 추상 메소드로 정의해서
서브클래스에서 오버라이드하여 새롭게 정의해 쓰도록 하는 것이다
이를 활용하면 다음과 같이 수정할 수 있을 것이다

```java
public class DeleteAllUserDao extends UserDao{
    public void deleteAll() throws SQLException{
        Connection c = null;
        PreparedStatement ps = null;

        try{
            c = dataSource.getConnection();
    //      ps = c.preparedStatement("delete from users");
            ps = makeStatement(c);
            ps.executeUpdate();
        }catch(SQLException e){
            throw e;
        }finally {
            if (c != null) {
                try{
                    c.close();
                } catch (SQLException e) {
                }
            }
            if (ps != null) {
                try{
                    ps.close();
                } catch(SQLException e){
                }
            }
        }
    }
    public abstract PreparedStatement makeStatement(Connection c) throws SQLException;
}
```
이제 업데이트와 관련된 메소드를 사용할 때마다 상속을 통해 makeStatement 메소드만 작성해 주면 될 것이다
이제 OCP의 원칙을 지키는 괜찮은 코드가 완성된 것 같다
하지만 이 코드에도 단점이 존재한다
가장 큰 문제는 DAO 로직마다 새로운 클래스를 만들어야 한다는 점이다
만약 로직이 4개라면 총 4개의 서브클래스를 만들어 사용해야 한다

그렇다면 이번엔 다른 방법으로 수정해 보자

## 전략 패턴의 적용
OCP를 잘 지키면서 템플릿 메소드 패턴보다 유연하고 확장성이 뛰어난 것이 바로 전략 패턴이다
전략 패턴이란, 오브젝트를 아예 둘로 분리하고, 클래스 레벨에서는 인터페이스를 통해서만 의존하도록 만드는 방법이다

이를 적용한 코드를 살펴보자

```java
public interface StatementStrategy {
  public PreparedStrategy makePreparedStrategy(Connection c) throws SQLException;
}

public class DeleteAllStatementStrategy implements StatementStrategy {
    @Override
    public PreparedStrategy makePreparedStrategy(Connection c) throws SQLException {
        return c.prepareStatement("delete from users");
    }
}

public class UserDao {
  public void deleteAll() throws SQLException {
    Connection c = null;
    PreparedStatement ps = null;

    try {
      c = dataSource.getConnection();

      StatementStrategy strategy = new DeleteAllStatementStrategy();
      ps = strategy.makePreparedStatement(c);

      ps.executeUpdate();
    } catch (SQLException e) {
      throw e;
    } finally {
      if (c != null) {
        try {
          c.close();
        } catch (SQLException e) {
        }
      }
      if (ps != null) {
        try {
          ps.close();
        } catch (SQLException e) {
        }
      }
    }
  }
}
```

전략 패턴을 적용함으로써 낮은 결합력을 갖게 되었다
하지만 이 코드에도 큰 문제가 있다
로직의 클라이언트인 UserDao에서 구체적인 전략인 DeleteAllStatementStrategy 클래스를 알고 있어야 한다
이는 OCP에도 맞지 않고 IoC에도 맞지 않다고 할 수 있다
이를 해결하려면 전략을 외부에서 주입하도록 하면 될 것이다

```java
public interface StatementStrategy {
  public PreparedStrategy makePreparedStrategy(Connection c) throws SQLException;
}

public class DeleteAllStatementStrategy implements StatementStrategy {
    @Override
    public PreparedStrategy makePreparedStrategy(Connection c) throws SQLException {
        return c.prepareStatement("delete from users");
    }
}

public class UserDao {

    public void deleteAll() throws SQLException{
        StatementStrategy strategy = new DeleteAllStatementStrategy();
        jdbcContextWithStatementStrategy(strategy);
    }

    public void jdbcContextWithStatementStrategy(StatementStrategy stmt) throws SQLException {
      Connection c = null;
      PreparedStatement ps = null;

      try {
        c = dataSource.getConnection();

        ps = stmt.makePreparedStatement(c);

        ps.executeUpdate();
      } catch (SQLException e) {
        throw e;
      } finally {
        if (c != null) {
          try {
            c.close();
          } catch (SQLException e) {
          }
        }
        if (ps != null) {
          try {
            ps.close();
          } catch (SQLException e) {
          }
        }
      }
    }
}
```

이제 동일한 전략으로 add 메서드를 작성해보자
```java
public interface StatementStrategy {
    public PreparedStrategy makePreparedStrategy(Connection c) throws SQLException;
}

public class AddStatementStrategy implements StatementStrategy {
    private User user;

    public AddStatementStrategy(User user) {
        this.user = user;
    }

    @Override
    public PreparedStrategy makePreparedStrategy(Connection c) throws SQLException {
        PreparedStatement ps = c.preparedStatement("insert into users(id, name, password) values(?,?,?)");
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());
        return ps;
    }
}

public class UserDao {

    public void add(User user) throws SQLException{
        StatementStrategy strategy = new AddStatementStrategy(user);
        jdbcContextWithStatementStrategy(strategy);
    }

    public void jdbcContextWithStatementStrategy(StatementStrategy stmt) throws SQLException {
      Connection c = null;
      PreparedStatement ps = null;

      try {
        c = dataSource.getConnection();

        ps = stmt.makePreparedStatement(c);

        ps.executeUpdate();
      } catch (SQLException e) {
        throw e;
      } finally {
        if (c != null) {
          try {
            c.close();
          } catch (SQLException e) {
          }
        }
        if (ps != null) {
          try {
            ps.close();
          } catch (SQLException e) {
          }
        }
      }
    }
}
```

아직 아쉬움이 남는 부분이 있다
모든 statement 마다 새로운 클래스를 생성해야 하고, 클래스 파일이 점점 늘어날 것이다
따라서 이를 해결하기 위해 클래스를 로컬 클래스로 변경해 보자


```java
public interface StatementStrategy {
    public PreparedStrategy makePreparedStrategy(Connection c) throws SQLException;
}

public class UserDao {
    public void add(Final User user) throws SQLException{
        public class AddStatementStrategy implements StatementStrategy {
            @Override
            public PreparedStrategy makePreparedStrategy(Connection c) throws SQLException {
                PreparedStatement ps = c.preparedStatement("insert into users(id, name, password) values(?,?,?)");
                ps.setString(1, user.getId());
                ps.setString(2, user.getName());
                ps.setString(3, user.getPassword());
                return ps;
            }
        }

        StatementStrategy strategy = new AddStatementStrategy(user);
        jdbcContextWithStatementStrategy(strategy);
    }
}
```

add 메서드 내부에 로컬 클래스로 선언하여 클래스 파일이 늘어나는 것을 방지했다
또한 파라미터로 전달된 User 클래스를 Final로 선언하므로써 UserDao 내부에 불필요한 User객체 선언을 방지할 수 있게 되었다

AddStatementStrategy 클래스는 UserDao의 add 메서드에서만 사용하기 때문에 익명 함수로 바꿀 수 있다
더 줄여서 표현하면 다음과 같다

```java
public interface StatementStrategy {
  public PreparedStrategy makePreparedStrategy(Connection c) throws SQLException;
}

public class UserDao {
  public void add(Final User user) throws SQLException {
    jdbcContextWithStatementStrategy(new StatementStrategy() {
        @Override
        public PreparedStrategy makePreparedStrategy(Connection c) throws SQLException {
          PreparedStatement ps = c.preparedStatement("insert into users(id, name, password) values(?,?,?)");
          ps.setString(1, user.getId());
          ps.setString(2, user.getName());
          ps.setString(3, user.getPassword());
          return ps;
        }
    });
  }
}
```

이제 UserDao 외부 클래스에서도 사용할 수 있도록 jdbcContextWithStatementStrategy 메서드를 외부로 분리해보자

```java
public class JdbcContext {
  private DataSource dataSource;

  public void setDataSource(DataSource dataSource) {
    this.dataSource = dataSource;
  }

  public void workWithStatementStrategy(StatementStrategy stmt) throws SQLException {
    Connection c = null;
    PreparedStatement ps = null;

    try {
      c = dataSource.getConnection();

      ps = stmt.makePreparedStatement(c);

      ps.executeUpdate();
    } catch (SQLException e) {
      throw e;
    } finally {
      if (c != null) {
        try {
          c.close();
        } catch (SQLException e) {
        }
      }
      if (ps != null) {
        try {
          ps.close();
        } catch (SQLException e) {
        }
      }
    }
  }
}

public class UserDao {

    private JdbcContext jdbcContext;

    public void setJdbcContext(JdbcContext jdbcContext) {
        this.jdbcContext = jdbcContext;
    }

    public void add(Final User user) throws SQLException {
        this.jdbcContext.workWithStatementStrategy(new StatementStrategy() {
            @Override
            public PreparedStrategy makePreparedStrategy(Connection c) throws SQLException {
              PreparedStatement ps = c.preparedStatement("insert into users(id, name, password) values(?,?,?)");
              ps.setString(1, user.getId());
              ps.setString(2, user.getName());
              ps.setString(3, user.getPassword());
              return ps;
            }
        });
    }
}
```

이제 외부에서 사용할 수 있도록 분리하여 DI를 적용하였다
하지만 DI를 적용할 때 인터페이스가 아닌 구체적인 클래스를 적용하였다
그렇다면 굳이 DI를 받도록 빈으로 등록할 필요가 있을까?
결론부터 말하면 그렇다
DI를 위해서는 주입하는 오브젝트와 주입받는 오브젝트 모두 스프링 컨테이너에 등록되어 있어야 하기 때문이다

일반적인 DI라면 템플릿에 인스턴스를 만들어 두고 사용할 오브젝트를 주입받아 사용할 것이다
하지만 우리가 작성한 코드는 매번 메소드 단위로 사용할 오브젝트를 전달받아 사용한다
이러한 패턴을 템플릿/콜백 패턴이라고 한다

이제 콜백 메서드를 추출해보자
delete와 같이 단일 쿼리로 이루어진 메서드가 사용하도록 메서드를 분리해보자

```java
public class UserDao{
    public void deleteAll() {
        executeQuery("delete from users");
    }

    public void executeQuery(String query) {
        this.jdbcContext.workWithStatementStrategy(new StatementStrategy() {
            @Override
            public PreparedStrategy makePreparedStrategy(Connection c) throws SQLException {
              PreparedStatement ps = c.preparedStatement(query);
              return ps;
            }
        });
    }

}
```
이제 deleteAll 메서드가 매우 간단해졌다
추출해낸 executeQuery 메서드는 UserDao만 가지고 있기에 아까우니까 JdbcContext로 이동해보자

```java
public class JdbcContext {
    public void executeQuery(String query) {
        this.jdbcContext.workWithStatementStrategy(new StatementStrategy() {
            @Override
            public PreparedStrategy makePreparedStrategy(Connection c) throws SQLException {
              PreparedStatement ps = c.preparedStatement(query);
              return ps;
            }
        });
    }
}

public class UserDao{
    public void deleteAll() {
        this.jdbcContext.executeQuery("delete from users");
    }
}
```
이제 모든 DAO 메서드에서 executeSql 메서드를 사용할 수 있게 되었다

## 템플릿/콜백의 응용
고정된 작업 흐름을 가지고 있으면서, 여기저기 반복되는 코드가 있다면, 중복되는 코드를 분리할 방법을 생각해보아야 한다
중복된 코드는 먼저 메소드롤 분리하는 간단한 시도를 해 본다
그중 필요에 따라 바꾸어야 하는 부분이 있다면, 인터페이스를 사이에 두고 분리해서 전략 패턴을 적용하고 DI로 의존관계를 관리한다
만약 바뀌는 부분이 여러 종류가 만들어진다면, 템플릿/콜백 패턴을 적용할 것을 고려할 수 있다


이번 포스팅에서 알아본 것을 정리하면 다음과 같다
- JDBC와 같이 예외 발생 가능성이 있으며 공유 리소스 반환이 필요한 코드는 반드시 try/catch/finally 블록으로 관리하자
- 일정한 작업 흐름이 반복되면서, 그중 일부 기능만 바뀌는 코드가 존재한다면 전략 패턴을 적용한다
- 변하지 않는 부분을 템플릿, 변하는 부분을 전략으로 만들고 인터페이스를 통해 유연하게 전략을 변경한다
- 같은 어플리케이션 내부에서 여러 종류의 전략을 다이나믹하게 구성하고 사용해야 한다면 컨텍스트를 이용하는 클라이언트 메소드에서 직접 전략을 정의해야 한다
- 클라이언트 메소드 안에 익명 내부 클래스를 사용하여 전략 오브젝트를 구성하면 편리하다
- 컨텍스트가 하나 이상의 클라이언트 오브젝트에서 사용된다면 클래스를 분리해서 공유하도록 한다
- **단일 전략 메소드를 갖는 전략 패턴이면서, 익명 내부 클래스를 사용해서 매번 전략을 다르게 가져가고, 컨텍스트 호출과 동시에 전략 DI를 수행하는 방식을 템플릿/콜백 패턴이라고 한다**
- 콜백의 코드에도 일정한 패턴이 반복된다면, 콜백을 템플릿에 넣고 재활용하도록 한다
- 템플릿과 콜백의 타입이 바뀔 수 있다면 제너릭스를 사용하도록 한다

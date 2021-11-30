---
title: 스프링 테스트(test)
date: 2021-11-30 00:00:01
categories: [Spring, Test]
tags: [spring, test, junit]     # TAG names should always be lowercase
---
어떤 기능을 테스트 할 때 고려할 사항이 몇 가지 존재한다
하나하나 살펴보자
## 작은 단위의 테스트

어떤 기능을 개발할 때 높은 응집도를 가지는 것이 좋다고 했다
테스트를 작성할 때에도 동일하게 관심사의 분리가 이루어져야 한다
테스트의 관심이 다르다면 테스트할 대상을 분리하고 집중해서 접근해야 한다

이렇게 작은 단위의 코드에 대해 테스트를 수행한 것을 단위 테스트(Unit Test)라고 한다
여기서 말하는 단위는 정해진 것이 아니며, 하나의 관심에 집중해 효율적으로 테스트 할 수 있는 범위의 단위로 보면 된다

## 자동수행 테스트

테스트를 진행할 때 웹 화면에 폼을 띄운 뒤 매번 값을 입력하고 클릭하는 과정을 반복하면 효율성이 저하될 것이다
자바 코드로만 작성되어 자동으로 수행되는 테스트의 경우 코드의 수정이 발생해도 빠르게 테스트를 실행할 수 있다

## 지속적 개선과 점진적 개발을 위한 테스트

각 기능들에 대해 테스트를 작성하면서 코드를 작성하면 나중에 설계 오류가 발견되는 상황에서도
어떤 부분에 문제가 발생했는지 확신을 가지고 찾아낼 수 있게 된다

또 기능을 추가하려고 해도 미리 만들어 놓은 테스트 코드는 유용하다
새로운 기능을 추가한다고 해도 기존에 만들어뒀던 기능들이 수정한 코드에 영향을 받지 않고 동작하는 것을 확인할 수 있다


## UserDaoTest의 문제점

여기 이전 spring IoC 를 학습하기 위해 작성한 테스트 코드가 있다

```java
public class UserDaoTest{
  public static void main(String[] args) throws SQLException{
      ApplicationConext = new GenericXmlApplicationContext("applicationContext.xml");
      UserDao dao = context.getBean("userDao", UserDao.class);

      User user = new User();
      user.setId("user");
      user.setName("안재홍");
      user.setPassword("developer");

      dao.add(user);

      System.out.println(user.getId() + " 등록 성공");

      User user2 = dao.get(user.getId());
      System.out.println(user2.getName());
      System.out.println(user2.getPassword());

      System.out.println(user2.getId() + " 조회 성공");
  }
}
```

이 테스트 방법은 main 메소드를 사용하였고, Application Context를 생성하여 UserDao 객체를 직접 호출했다
하지만 이 코드에는 여러 문제점들이 존재한다

### 수동 확인 작업의 번거로움
위 코드에서 입력과 출력 과정은 자동으로 진행되었으나 그 값을 검증하는 것은 개발자의 몫이다

### 실행 작업의 번거로움
만약 테스트의 개수가 증가하면, 단순히 main메소드로 확인하기에는 번거로움이 발생한다


## UserDaoTest 개선

UserDaoTest의 두 가지 문제점을 개선해 보자

### 테스트 검증의 자동화

첫 번째 문제점인 테스트 결과의 검증 부분을 코드로 만들어 보자

```java
public class UserDaoTest{
  public static void main(String[] args) throws SQLException{
      ApplicationConext = new GenericXmlApplicationContext("applicationContext.xml");
      UserDao dao = context.getBean("userDao", UserDao.class);

      User user = new User();
      user.setId("user");
      user.setName("안재홍");
      user.setPassword("developer");

      dao.add(user);

      System.out.println(user.getId() + " 등록 성공");

      User user2 = dao.get(user.getId());
      System.out.println(user2.getName());
      System.out.println(user2.getPassword());

      // System.out.println(user2.getId() + " 조회 성공");
      if (!user.getName().equals(user2.getName())) {
          System.out.println("테스트 실패 (name)");
      }
      else if (!user.getPassword().equals(user2.getPassword())) {
          System.out.println("테스트 실패 (password)");
      } else {
          System.out.println("조회 테스트 성공");
      }
  }
}
```

이제 검증이 완벽히 자동화 되었다
테스트를 실행하고 단순히 성공 메시지만 확인하면 될 것이다

이제 다음 문제를 해결해 보자
main 메소드가 아닌 JUnit 테스팅 프레임워크는 일정한 패턴을 가진 테스트를 만들 수 있고, 많은 테스트를 간단히 실행시킬 수 있으며,
테스트 결과를 종합적으로 볼 수 있고, 테스트가 실패한 곳을 빠르게 찾을 수 있다

### Junit 테스트로 전환

기존의 main 메소드는 제어권을 직접 갖는다
JUnit은 테스팅을 위한 프레임워크고 프레임워크는 제어 권한을 넘겨받아 주도적으로 애플리케이션 흐름을 제어한다
따라서 JUnit을 사용하면 main 메소드도 필요 없고 오브젝트를 만들어 실행할 필요도 없어진다

이제 main 메소드를 제거하고 Junit의 메소드로 변경해 보자
Junit은 테스트 메소드가 따라야 할 조건을 두 가지 설정해 놓았는데
첫 번째는 public으로 선언되어야 하는 것이고, 다른 하나는 @Test라는 어노테이션을 붙여주는 것이다
이를 만족하도록 코드를 수정해 보겠다

```java
public class UserDaoTest{
  @Test
  public void addAndGet() throws SQLException{
      ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

      UserDao dao = context.getBean("userDao", UserDao.class);
      ...
  }

}
```

이제 테스트 코드의 결과를 검증하는 if/else 문을 Junit이 제공하는 방법으로 전환해 보겠다

```java
public class UserDaoTest{
  @Test
  public void addAndGet() throws SQLException{
      ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

      UserDao dao = context.getBean("userDao", UserDao.class);
      User user = new User();
      user.setName("안재홍");
      user.setId("koji4321");
      user.setPassword("springno1");

      dao.add(user);

      User user2 = dao.get(user.getId());

      assertThat(user2.getName(), is(user.getName()));
      assertThat(user2.getPassword(), is(user.getPassword()));
  }

}
```

이렇게 JUnit을 이용하도록 테스트 코드를 수정하였다
하지만 이 코드에도 문제가 있다
UserDaoTest의 문제는 이전 테스트 때문에 DB에 등록된 중복 데이터가 있을 수 있다는 점이다
가장 좋은 해결책은 addAndGet() 테스트를 마치고 나면 테스트가 등록한 사용자 정보를 삭제하는 것이다
User 테이블의 모든 레코드를 삭제하는 deleteAll 메소드를 작성해 보자

```java
public class UserDao{
    public void deleteAll() throws SQLException {
        Connection c = dataSource.getConnection();

        PreparedStatement ps = c.preparedStatement("delete from users");
        ps.executeUpdate();

        ps.close();
        c.close();
    }
}
```
그리고 삭제 확인을 위해 레코드 개수를 확인하는 getCount 메소드를 작성해 보자
```java
public class UserDao{
    public int getCount() throws SQLException {
        Connection c = dataSource.getConnection();

        PreparedStatement ps = c.preparedStatement("select count(*) from users");
        ResultSet rs = ps.executeQuery();
        rs.next();
        int count = rs.getInt(1);

        rs.close();
        ps.close();
        c.close();
    }
}
```

이제 테스트 코드에서 작성한 기능들을 사용해보자

```java
public class UserDaoTest{
  @Test
  public void addAndGet() throws SQLException{
      ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
      UserDao dao = context.getBean("userDao", UserDao.class);

      dao.deleteAll();
      assertThat(dao.getCount(), is(0));

      User user = new User();
      user.setName("안재홍");
      user.setId("koji4321");
      user.setPassword("springno1");

      dao.add(user);
      assertThat(dao.getCount(), is(1));

      User user2 = dao.get(user.getId());

      assertThat(user2.getName(), is(user.getName()));
      assertThat(user2.getPassword(), is(user.getPassword()));
  }

}
```

이제 몇 번을 실행하더라도 동일한 결과를 보장할 수 있게 되었다
하지만 만약, get 메서드의 입력값에 존재하지 않는 id값이 입력되면 어떻게 될까
추가적으로 get 메서드의 입력값에 대한 예외조건을 처리하기 위해 @Test 어노테이션의 파라미터를 지정해 줄 수 있다
테스트 중에 발생할 것으로 기대하는 예외 클래스를 지정하면 정상적으로 동작할 것이다
또한 get 메서드에서도 예외가 발생할 것이므로 이를 밖으로 던져 주도록 수정해야 할 것이다

```java
public class UserDaoTest{
  @Test(expected=EmptyResultDataAccessException.class)
  public void getUserFailure() throws SQLException{
      ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
      UserDao dao = context.getBean("userDao", userDao.class);

      dao.deleteAll();
      assertThat(dao.getCount(), is(0));

      dao.get("unknown_id");    // 오류 발생 지점
  }
}

public class UserDao{
    public User get() throws SQLException {
        ...
        ResultSet rs = ps.executeQuery();

        User user = null;
        if (rs.next()) {
            user = new User();
            user.setName(rs.getString(id));
            user.setName(rs.getString(name));
            user.setName(rs.getString(password));
        }

        rs.close();
        ps.close();
        c.close();

        if (user == null) {
            throw new EmptyResultDataAccessException(1);
        }

        return user;
    }
}
```

이제 정말 정상적으로 동작되는 코드가 작성되었다
이제 JUnit 프레임워크가 제공하는 다양한 기능들을 사용하여 성능을 향상시키고, 단순화를 시켜 보자

먼저 UserDaoTest의 코드를 살펴 보면, 중복되는 코드가 존재한다
ApplicationContext를 생성하고 UserDao 빈을 받아오는 과정이 중복된다
JUnit에는 중복된 과정을 생략할 수 있도록 @Before 어노테이션을 제공한다
이를 활용하면 다음과 같이 작성할 수 있다

```java
public class UserDaoTest{
    private UserDao dao;

    @Before
    public void setUp() {
      ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
      dao = context.getBean("userDao", UserDao.class);
    }

    @Test
    public void addAndGet() throws SQLException{
        User user = new User();
        user.setName("안재홍");
        user.setId("koji4321");
        user.setPassword("springno1");

        dao.add(user);

        User user2 = dao.get(user.getId());

        assertThat(user2.getName(), is(user.getName()));
        assertThat(user2.getPassword(), is(user.getPassword()));
    }

}
```

JUnit이 하나의 테스트 클래스를 가져와 테스트를 수행하는 방식은 다음과 같다

1. 테스트 클래스에서 @Test가 붙은 public이고 void형이며 파라미터가 없는 메서드를 모두 찾는다
2. 테스트 클래스의 오브젝트를 하나 만든다
3. @Before이 붙은 메서드를 실행한다
4. @Test가 붙은 메소드를 하나 호출하고 테스트 결과를 저장한다
5. @After이 붙은 메서드가 있으면 실행한다
6. 나머지 메서드에 대해 2~5를 반복한다
7. 테스트 결과를 종합해 돌려준다

테스트 코드에서 필요한 정보나 오브젝트를 픽스쳐(Fixture)라고 한다
일반적인 픽스쳐는 여러 테스트에서 반복 사용되므로 @Before 메서드를 이용해 생성해 두면 편하다
UserDaoTest 클래스에서 픽스쳐는 UserDao, User가 해당된다
이제 해당 픽스쳐를 미리 선언해놓고 @Before 메서드를 통해 초기화를 시켜 보자


```java
public class UserDaoTest{
    private UserDao dao;
    private User user1;
    private User user2;
    private User user3;


    @Before
    public void setUp() {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        dao = context.getBean("userDao", UserDao.class);

        this.user1 = new User("koji", "안재홍", "spingno1");
        this.user2 = new User("panda", "오승주", "kubernetesno1");
        this.user3 = new User("bdj", "배동준", "displayno1");
    }

    @Test
    public void addAndGet() throws SQLException{
        dao.add(user1);

        User findUser = dao.get(user1.getId());

        assertThat(findUser.getName(), is(user.getName()));
        assertThat(findUser.getPassword(), is(user.getPassword()));
    }

}
```

이제 ApplicationContext의 생성 방식을 변경해 보자
지금까지는 매 테스트마다 ApplicationContext를 생성했다
각 테스트는 독립된 상황에서 독립적인 오브젝트를 사용하는 것이 원칙이나, 생성에 시간과 자원이 많이 소모되는 경우,
테스트 전체가 공유하는 오브젝트를 만들기도 한다
ApplicationContext는 초기화된 후 변경되는 일이 없고, 각 빈은 싱글톤 방식으로 생성되어 stateless 상태이다
따라서 이 ApplicationContext를 여러 테스트에서 공유하도록 작성할 수 있다

```java

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "/applicationContext.xml")
public class UserDaoTest {
    @Autowired
    private ApplicationContext context;

    ...

    @Before
    public void setUp() {
        this.dao = this.context.getBean("userDao", UserDao.class);
    }
}
```

@ContextConfiguration 어노테이션에 지정한 설정 파일을 읽어서 초기화하는 과정을 진행했다
하지만 우리가 작성한 applicationContext.xml에는 ApplicationContext에 관련된 빈이 존재하지 않는데 어떻게 된 것일까
ApplicationContext는 초기화할 때 자기 자신도 빈으로 등록한다
따라서 해당 빈이 존재하고 DI도 가능해진 것이다

그렇다면 굳이 ApplicationContext를 DI받지 않고, UserDao빈을 DI받으면 되지 않을까?

```java

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "/applicationContext.xml")
public class UserDaoTest {
    @Autowired
    private UserDao dao;

    ...

    @Before
    public void setUp() {
        ...
    }
}
```

이렇게 DI가 완료된 UserDao를 가지게 되었다
여기서 @Autowired는 변수에 할당 가능한 타입을 가진 빈을 자동으로 찾는다
만약 타입이 여러 개인 경우 CamelCase를 적용하여 동일한 이름을 가진 빈을 주입한다

이제 얼추 해결 된 듯 보이지만, 아직 문제가 남아 있다
지금까지 사용한 DB가 운영용 DB라고 생각하면, 이를 가지고 테스트를 진행할 수는 없을 것이다
따라서 새로운 테스트용 DB를 사용하는 것이 필요한데, 이는 새로운 컨테이너 설정 파일을 사용하는 것으로 해결이 가능하다

```java

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "/test-applicationContext.xml")
public class UserDaoTest {
  ...
}
```

이번 포스팅에서 알아본 것들을 총 정리하면 다음과 같다

- 테스트는 자동화되어야 하고, 빠르게 실행할 수 있어야 한다
- main()테스트 대신 JUnit 프레임워크를 이용한 테스트 작성이 편리하다
- 테스트 결과는 일관성이 있어야 한다. 코드의 변경 없이 환경이나 실행 순서에 무관하게 실행되어야 한다
- 테스트는 포괄적으로 작성해야 한다. 충분한 검증을 하지 않는 테스트는 없는 것보다 나쁘다
- 코드 작성과 테스트 수행 사이의 간격이 짧을수록 효과적이다
- 테스트하기 쉬운 코드가 좋은 코드다
- 테스트를 만들고 테스트를 성공시키도록 코드를 작성하는 TDD도 유효한 개발방법이다
- 테스트 코드 또한 리팩토링이 필요하다
- @Before, @After를 사용해서 테스트 메소드들의 공통 준비 작업과 정리 작업을 처리할 수 있다
- 스프링 테스트 컨텍스트 프레임워크를 사용하면 성능을 향상할 수 있다
- 동일한 설정파일을 사용하는 테스트는 하나의 ApplicationContext를 공유한다

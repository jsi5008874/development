  

프로젝트의 규모가 커진다면 그만큼 테스트할 기능도 많아지게 된다.

그렇다면 많은 테스트를 간단히 실행시키고, 테스트 결과를 종합해서 볼 수 있고, 테스트가 실패한 곳을 빠르게 찾을 수 있는 기능을 갖춘 테스트 지원 도구가 필요하다.

  

그것이 바로 **java의 JUnit**이다.

  

### JUnit 테스트로 전환

JUnit도 프레임워크이다. IOC를 기반으로 작동하므로 개발자가 만든 클래스에 대한 제어 권한을 넘겨받아 주도적으로 흐름을 제어한다.

따라서 UserDaoTest처럼 main() 메소드도 필요없고 오브젝트를 만들어서 실행시키는 코드를 만들 필요도 없다.

  

테스트 메소드는 JUnit이 요구하는 두 가지 조건을 따라야한다.

첫 째는 public으로 선언되어야하고 두 번째는 @Test라는 어노테이션을 붙여줘야한다.

  

```Java
public class UserDaoTest{
  @Test
  public void addAndGet() throws SQLException {
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    
    UserDao dao = context.getBean("userDao", UserDao.class);
    .....
  }
}
```

  

### 검증 코드 전환

if/else문으로 했던 검증을 JUnit에서 제공하는 방법을 이용해 전환해보자

  

if 문장의 기능을 JUnit이 제공해주는 assertThat이라는 스태틱 메소를 이용해서 변경한다.

```Java
if(!user.getName().equals(user2.getName()).....

//JUnit으로 변경
assertThat(user2.getName(), is(user.getName()));
```

  

assertThat() 메소드는 첫 번째 파라미터의 값을 뒤에 나오는 매처(Matcher)라고 불리는 조건으로

비교해서 일치하면 다음으로 넘어가고 아니면 테스트가 실패하도록 만들어준다.

is()는 매처의 일종으로 equals()와 동일한 기능을 가졌다.

  

JUnit은 예외가 발생하거나 asserThat()에서 실패하지 않고 테스트 메소드의 실행이 완료되면

테스트가 성공했다고 인식한다..

  

### **Junit 적용**

```Java
@SpringBootTest
@ContextConfiguration(classes = DaoFactory.class)
public class UserDaoTest {
    @Autowired
    UserDao userDao;

    @Test
    public void addAndGet() throws SQLException, ClassNotFoundException {
        User user = new User();
        user.setId("gyumee");
        user.setName("박성철");
        user.setPassword("springno1");

        userDao.add(user);

        User user2 = userDao.get(user.getId());

        assertThat(user2.getName()).isEqualTo(user.getName());
        assertThat(user2.getPassword()).isEqualTo(user.getPassword());
    }
}
```

SpringBoot 기준으로 작성한 코드이다.

  

@SpringBootTest로 테스트 클래스임을 명시하고  
@ContextConfiguration으로 DaoFactory를 애플리케이션 컨텍스트에 등록한다.  

  

이후 @Autiwored로 userDao를 주입하면 자동으로 DaoFactory가 UserDao를 생성해서 주입한다.

  

테스트할 메서드에 @Test를 붙여서 테스트를 진행한다.

  

![[image/image 361.png|image 361.png]]

테스트 성공하면 이렇게 표시된다.

  

또한 테스트는 여러 클래스를 한번에 할 수도 있다.

  

![[image/image 1 2.png|image 1 2.png]]

테스트를 패키지 단위로 실행하면 패키지 하위에 있는 모든 테스트가 동시에 실행되고 결과가 출력된다.

  

### 테스트 결과의 일관성

바로 위의 사진을 보면 UserDaoTest의 결과를 보면 X로 실패 표시가 되어있다.

이유는 이전에 UserDaoTest를 실행하면서 DB에 데이터가 삽입되었는데 다시 테스트를 돌리면서

동일한 데이터가 DB 삽입 시도되어서 primary key 중복 오류가 발생한 것이다.

  

**이처럼 외부의 요인으로 인해 테스트가 실패한다면 좋은 테스트는 아니다**

**코드에 변경사항이 없다면 항상 동일한 결과를 내야 한다.**

  

**DeleteAll()의 getCount() 추가**

```Java
public void deleteAll() throws ClassNotFoundException, SQLException{
        Class.forName("com.mysql.cj.jdbc.Driver");

        PreparedStatement ps = connectionMaker.makeConnection().prepareStatement(
                "delete from users");
        
        ps.executeUpdate();
        
        ps.close();
        
        connectionMaker.makeConnection().close();
    }
```

UserDao에 deleteAll 메서드 추가

users 테이블의 모든 데이터를 삭제하는 메서드이다.

  

```Java
public int getCount() throws ClassNotFoundException, SQLException{
        Class.forName("com.mysql.cj.jdbc.Driver");

        PreparedStatement ps = connectionMaker.makeConnection().prepareStatement(
                "select count(*) from users");
        
        ResultSet rs = ps.executeQuery();
        rs.next();
        int count = rs.getInt(1);
        
        rs.close();
        ps.executeUpdate();

        ps.close();

        connectionMaker.makeConnection().close();
        
        return count;
    }
```

getCount()메서드로 users 테이블의 레코드 개수를 반환해주는 메서드이다.

  

새로 만든 두 메서드를 Test 코드에서 활용한다.

  

deleteAll()로 데이터 삽입 전 모든 레코드를 삭제한 후 삽입 테스트를 실행하면 중복오류 없이 정상적으로

실행 될 것이다.

getCount()로 삭제 후 레코드 수 확인, 삽입 후 레코드 수 확인해서 이상유무를 판단한다.

  

```Java
@Test
    public void addAndGet() throws SQLException, ClassNotFoundException {
        userDao.deleteAll();
        assertThat(userDao.getCount()).isEqualTo(0);

        User user = new User();
        user.setId("gyumee");
        user.setName("박성철");
        user.setPassword("springno1");

        userDao.add(user);
        assertThat(userDao.getCount()).isEqualTo(1);

        User user2 = userDao.get(user.getId());

        assertThat(user2.getName()).isEqualTo(user.getName());
        assertThat(user2.getPassword()).isEqualTo(user.getPassword());
    }
```

해당 테스트를 돌려보면

  

![[image/image 2 2.png|image 2 2.png]]

이상 없이 테스트가 진행된 것을 볼 수 있다.

  

**이처럼 단위 테스트는 항상 일관성을 보장해야한다.**

**외부의 영향을 받지 않고 코드가 변경되지 않는 한 테스트 결과가 동일해야한다.**

  

### 예외조건 추가

get() 메소드에 전달된 id 값에 해당하는 사용자의 정보가 없다면?

두 가지 방법이 있는데 하나는 null을 리턴하는 것이고, 하나는 예외를 던지는 것이다.

  

이 때 주어진 id에 해당하는 정보가 없다는 의미를 가진 예외 클래스를 지정해서 사용한다.

SQLException 예외를 활용하여 테스트를 해보겠다.

(각 상황에 맞는 Exception을 사용하면 된다. 현재는 DB에서 데이터를 가져오는 동안 null인 경우를

테스트 하기 위해 위 의 Exception을 사용했다.)

  

일반적인 상황에서는 테스트 도중 Exception이 발생하면 검증 실패이지만

현재의 상황에서는 Exception이 발생해야 검증 성공이다.

  

```Java
@Test
    public void getUserFailure() throws SQLException, ClassNotFoundException{
        userDao.deleteAll();
        assertThat(userDao.getCount()).isEqualTo(0);

        userDao.get("unknown");
    }
```

우선 테이블의 모든 데이터를 지우고 id를 기준으로 조회를 한다.

![[image/image 3 2.png|image 3 2.png]]

SQLExeption이 발생한 것을 볼 수 있다.

  

### 예외상황에 대한 테스트

  

```Java
@Test
    public void getUserFailure() throws SQLException, ClassNotFoundException{
        userDao.deleteAll();
        assertThat(userDao.getCount()).isEqualTo(0);

        assertThrows(SQLException.class, ()-> userDao.get("id"));

    }
```

asserThrows()를 사용하면 예외가 발생해도 테스트 성공으로 표시된다.

  

![[image/image 4 2.png|image 4 2.png]]

  

assertThrows() 자체가 예외가 발생했을 때를 가정해서 테스트를 하는 메서드여서

예외가 발생하면 테스트 성공, 발생하지 않으면 테스트 실패로 출력된다.

  

### 포괄적인 테스트

  

간단한 기능이라도 여러 방향에서 테스트를 해보는 것이 중요하다.

이전에 DB에 insert하는 간단한 기능이더라도 userDao.add(), userDao.get(), userDao.getCount() 등

여러 방향으로 테스트를 했다.

**이렇게 하는 이유는 정상적으로 잘 작동하는것 처럼 보여도 특별한 상황에서 엉뚱하게 동작하는 경우가 있고**

**이런 경우 원인을 찾기 힘들기 때문이다.**

  

**그리고 개발자가 많이 하는 실수가 성공하는 테스트만 골라서 만드는 것이다.**

개발자는 잘 돌아가는 케이스만 상상해서 만드는데 테스트를 할 때도 그런 경향이 있다.

문제가 될 만한 상황이나 입력 값등은 교묘히 피해서 만드는 습성이 있다.

**그래서 테스트는 부정적인 케이스를 먼저 만들어서 테스트 해야한다.**

ex) userDao.get()을 테스트 할 때 null 값을 가져온 경우를 테스트

  

### 기능설계를 위한 테스트

getFailure() 테스트를 보면 기능설계 > 구현 > 테스트 라는 일반적인 흐름으로 진행한게 아니라

존재하지 않는 id로 get() 메소드를 싱행하면 특정한 예외가 던져져야 한다는 식으로 기능을 결정했다.

그리고 userDao의 코드를 수정하는 것보다 getUserFailure() 테스트를 먼저 만들었다.

이처럼 테스트 코드를 먼저 만들어서 개발을 진행하는 방법도 실제로 존재한다.

  

또한 getUserFailure()는 만들고 싶은 기능의 조건, 행위, 결과에 대한 내용이 잘 표현되어 있다.

사진

  

**이 테스트 코드는 마치 잘 작성된 하나의 기능서처럼 보이며 이런식으로 추가하고 싶은 기능을 테스트 코드로 표현해서 코드로 된 기능 설계서라고 생각하면 된다.**


### 테스트 주도 개발
방금 말한 것과 같이 기능의 내용을 담고 있으면서 만들어진 코드를 검증해주는 테스트 코드를 먼저 만들고
테스트를 성공하게 해주는 코드를 작성하는 방식을 테스트 주도 개발(TDD, Test Driven Development)라 한다.

언뜻 보면 개발 속도가 느릴것 같지만 실질적으로는 개발 속도를 빠르게 만들어주는 효과가 있다.
이유는 테스트 코드가 애플리케이션 코드보다 상대적으로 작성하기 쉽고 독립적이기 때문에 코드의 양에 비해
작성하는데 얼마 걸리지 않고 테스트 덕분에 오류를 빨리 잡아내기 때문이다.

일반적으로 서버에 올려서 실행하고 실패하면 다시 재시작하는 방식보다 일찍이 테스트 코드를 활용하면
시간적 이점이 생긴다.

### 테스트 코드 개선
테스트 코드도 리팩토링을 할 수 있다.
더 간결하고 깔끔하게 작성을 하되 테스트 결과가 일정하게 유지되도록 해야 한다.

@beforeEach, @AfterEach 를 사용해서 반복되는 코드를 줄여준다.

```java
userDao.deletAll();
assertThat(userDao.getCount()).isEqualTo(0);
```
해당 코드가 테스트 메서드 마다 반복되고 있다.
그리고 deleteAll()은 테스트 메서드의 시작 부분에서 실행되도록 구성되어 있다.

반복된 코드를 줄이기 위해 @BeforeEach 어노테이션을 사용한다.

```java
@BeforeEach
public void setUp(){
	userDao.deletAll();
	assertThat(userDao.getCount()).isEqualTo(0);
}
```
이렇게 테스트하는 클래스에 반복된 코드를 메서드에 넣어주고 @BeforeEach 어노테이션을 붙여주면
테스트 클래스 내의 모든 테스트 메서드 시작 전에 @BeforeEach 메서드를 먼저 실행하고 테스트를 실행한다.

**JUnit이 테스트를 수행하는 방식**
1.  테스트 클래스에서 @Test가 붙은 public void에 파라미터가 없는 테스트 메소드를 모두 찾는다.
2. 테스트 클래스의 오브젝트를 하나 만든다.
3. @BeforeEach이 붙은 메서드가 있으면 실행한다.
4. @Test가 붙은 메서드를 하나 호출하고 테스트 결과를 저장해둔다.
5. @AfterEach가 붙은 메서드가 있으면 실행한다.
6. 나머지 테스트 메소드에 대해 2~5번을 반복한다.
7. 모든 테스트 결과를 종합해서 돌려준다.

이처럼 테스트 메서드를 실행하기 전 후로 @BeforeEach, @AfterEach를 찾아서 실행한다.
그리고 특이한 점은 테스트 클래스를 인스턴스화한 후 재활용하지 않고 테스트 메서드 실행마다
새로 생성한다는 점이다.
이는 Junit 개발자가 모든 테스트가 독립적으로 실행되도록 하기 위해 의도 했기 때문이다.

### 픽스처
테스트를 수행하는데 필요한 정보나 오브젝트를 픽스처(fixture)라 한다.
일반적으로 픽스처는 여러 테스트에서 반복적으로 사용되기 때문에 @BeforeEach로 생성해두면 편하다.

실습에서 활용한 UserDaoTest에서는 User 객체가 픽스처가 된다.

```java
public class UserDaoTest{
	private UserDao userDao;
	private User user1;
	private User user2;
	private User user3;

@beforeEach
public void setUp(){
	this.user1 = new User("1", "11", "111");
	this.user2 = new User("2", "22", "222");
	this.user3 = new User("3", "33", "333");
}
}
```
이렇게 픽스처를 만들어두면 모든 테스트를 실행할 때마다 User 객체 3개를 가지고 시작하게 된다.

이렇게 공통적으로 사용되는 객체나 데이터의 코드를 한곳에 몰아서 리팩토링 할 수 있다.


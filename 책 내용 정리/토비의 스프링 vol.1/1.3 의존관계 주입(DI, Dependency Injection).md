  

### 의존관계

의존하고 있다는 뜻은 어떤 객체가 다른 객체에 영향을 미친다는 뜻이다

예를 들어 Class A가 Class B를 의존하고 있는 관계라면

B가 변경이 되면 A의 결과에도 영향을 미치게 된다.

ex) A에서 B에 정의된 메소드를 호출해서 사용중인데 B의 메소드에 변경이 생긴다면

A의 결과에도 변경이 생기게 된다.

  

**여기서 중요한 점은 의존 관계는 방향성이 있다는 것이다.**

A >> B로 의존하고 있다면 A는 B에 의해 영향을 받지만 B는 A에 의해 영향을 받지 않는다.

이 말은 A는 B가 변경되면 영향이 미치지만 B는 A가 변경되더라도 변화가 없다.

  

### 런타임 의존관계 설정

  

![[KakaoTalk_20250323_170500901.jpg]]

우선 설계 상의 의존관계를 보면 실제 객체를 사용할 클라이언트인 UserDao는

인터페이스인 ConnectionMaker를 의존한다.

그리고 ConnectionMaker의 구현체인 DConnectionMaker가 있는데 DConnectionMaker는

UserDao가 실제로 사용하게되는 오브젝트이다.

이렇게 실제 사용되는 오브젝트(예시에서는 DConnectionMaker)를 의존 오브젝트라 한다.

  

이런 경우 런타임 시 UserDao가 사용할 오브젝트는 어떤 클래스로 만든 오브젝트인지 알 수 없다.

**실제 UserDao 클래스의 코드상에는 DConnectionMaker에 대한 내용은 없고 ConnectionMaker만 있기 때문에 어떤 오브젝트를 사용할지 모르는 것이다.**

  

**그래서 애플리케이션 컨텍스트나 빈 팩토리 같은 제 3의 존재가 런타임 시점에 의존관계를 결정한다.**

  

### 의존관계 검색과 주입

의존관계 검색은 **자신이 필요로 하는 오브젝트를 능동적으로 찾는것**을 말한다.

이러면 IOC라고 할 수 없을 것이다.

하지만 자세히 보면 런타임 시 의존관계를 맺을 오브젝트를 결정하는 것과 생성 작업은 외부의

컨테이너에게 맡기지만 이를 가져올 때 메소드나 생성자를 통한 주입 대신 스스로 컨테이너에게

요청하는 방법이다.

  

ex)

```Java
public UserDao(){
  DaoFactory daoFactory = new DaoFactory();
  this.connectionMaker = daoFactory.connectionMaker();
}
```

이렇게 해도 UserDao는 여전히 자신이 어떤 ConnectionMaker 오브젝트를 사용할지 모른다.

여전히 코드의 의존 대상은 ConnectionMaker 인터페이스뿐이다.

(정확히 사용할 구현체가 DConnectiomMaker인지 MConnectionMaker인지 알 수 없다는 뜻)

  

**런타임 시에 DaoFactory가 만들어서 돌려주는 오브젝트와 다이나믹하게 런타임 의존관계를 가진다.**

  

따라서 IOC 개념을 잘 따르고 있는 코드이며 단지 적용 방법이 외부로부터의 주입이 아니라

스스로 IOC 컨테이너에게 요청을 하고 있는것이다.

  

**하지만 의존관계 검색보다는 의존관계 주입으로 구현하는 편이 더 깔끔하다.**

의존관계 검색은 코드안에 오브젝트 팩토리 클래스나 스프링 API가 나타난다.

이렇게 되면 사용자에 대한 DB 정보를 어떻게 가져올 것인가에 집중해야하는 UserDao 클래스에서

다른 성격의 코드들이 섞여버리게 된다.

따라서 보통은 의존관계 주입으로 구현하는 편이 좋다.

  

의존관계 검색은 애플리케이션 시작 시점에 적어도 한번은 사용해야한다.

애플리케이션 시작 시점에서는 DI를 이용해 오브젝트를 주입 받을 방법이 없기 때문이다.

  

**또 다른 차이점은 의존관계 검색은 검색하는 오브젝트 자신이 스프링 빈일 필요가 없다는 점이다.**

예시의 UserDao가 스프링 빈이 아니어도 필요한 오브젝트를 요청할 수 있다.

요청을 받는 ConnectionMaker만 빈이어도 상관업는 것이다. 스프링에서 요청을 하는 UserDao에게

원하는 ConnectionMaker를 건네주려면 ConnectionMaker의 정보만 스프링이 가지고 있으면 되기 때문이다.

  

반면 DI 방식이라면 둘 다 스프링 빈이어야 가능하다.

스프링에서 의존관계 주입을 하려면 요청한 쪽 받는 쪽 모두의 정보를 가지고 있어야 가능하기 때문이다.

  

### 메소드를 이용한 의존관계 주입

  

수정자(setter) 메소드를 이용한 주입

수정자 메소드는 외부에서 오브젝트 내부의 속성 값을 변경하려는 용도로 주로 사용된다.

수정자 메소드의 핵심기능은 파라미터로 전달된 값을 내부의 인스턴스 변수에 저장하는 것

외부로부터 받은 오브젝트 레퍼런스를 저장해뒀다가 내부의 메소드에서 사용하게 하는 DI 방식

  

```Java
private ConnectionMaker connectionMaker;

public void setConnectionMaker(ConnectionMaker connectionMaker){
   this.connectionMaker = connectionMaker;
}
```

이처럼 set 수정자 메소드를 활용하여 ConnectionMaker를 UserDao에 DI 하도록 설정했다.

  

```Java
@Bean
public UserDao userDao(){
   UserDao userDao = new UserDao();
   userDao.setConnectionMaker(connectionMaker());
   return userDao;
}
```

의존관계를 주입하는 시점과 방법만 달라졌을 뿐 결과는 동일하다.

  

일반 메소드를 이용한 주입

수정자 메소드는 한 번에 한 개의 파라미터만 가질 수 있지만 일반 메소드는 여러 파라미터를

가질 수 있다는 장점이 있다.
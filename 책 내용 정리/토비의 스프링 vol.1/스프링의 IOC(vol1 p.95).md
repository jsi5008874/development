  

스프링이 제어권을 가지고 직접 만들고 관계를 부여하는 오브젝트를 빈(Bean)이라 부른다

스프링 빈은 스프링 컨테이너가 생성과 관계설정, 사용 등을 제어해주는 제어의 역전이 적용된 오브젝트

  

스프링에서 빈의 생성과 관계설정 같은 제어를 담당하는 IOC 오브젝트를 빈 팩토리라 부른다

빈 팩토리보다는 더 확장된 애플리케이션 컨텍스트를 주로 사용한다.(두 가지는 거의 비슷)

  

**빈팩토리 : 빈을 생성하고 관계를 설정하는 IOC 기능에 초점을 맞춘 것**

**애플리케이션 컨텍스트 : 애플리케이션 전반에 걸쳐 모든 구성요소의 제어를 담당하는 IOC 엔진**

  

애플리케이션 컨텍스트는 별도의 설정정보를 가져와 이를 활용하여 빈의 생성, 관계설정 등의

제어 작업을 총괄

  

스프링이 설정정보로 인식하게 만들기 위해서 어노테이션을 사용한다

@Configuration을 사용하면 스프링이 오브젝트 설정을 담당하는 클래스라고 인식하게 되고

@Bean을 사용하면 빈을 생성하는 메소드로 인식한다.

```Java
@Configuration //애플리케이션 컨텍스트 또는 빈 팩토리가 사용할 설정정보라는 표시
public class DaoFactory{
  @Bean
  public UserDao userDao(){
    return new UserDao(connectionMaker());
    }
  @Bean
  public ConnectionMaker connectionMaker(){
    return new DConnectionMaker();
    }
  }    
```

  

이제 해당 설정정보들을 어플리케이션 컨텍스트에 적용하려면

AnnotationConfigApplicationContextfmf 사용하면 된다

  

```Java
ApplicationContext context =
  new AnnotationConfigApplicationContext(DaoFactory.class);
  UserDao dao = context.getBean("userDao", UserDao.class);
```

애플리케이션 컨텍스트를 만들 때 DaoFactory 클래스를 넣어주고 getBean()이라는 메소드를 통해

UserDao 오브젝트를 가져온다

  

getBean의 파라미터 중 “userDao”가 빈의 이름이고 빈의 이름은 @Bean으로 지정된 메소드의 이름이다

따라서 @Bean이 설정된 userDao() 메서드명이 빈의 이름이 되는 것이다

  

getBean의 두 번째 파라미터인 UserDao.class는 리턴되는 오브젝트의 타입을 지정해주는 것이다.

**최종적으로 getBean의 결과는 userDao() 메서드를 이용해 UserDao 타입의 오브젝트를 가져온다.**

  

### 애플리케이션 컨텍스트의 동작방식

IOC 설정정보를 이용하여(@Configuration같은 설정정보들) 객체간 관계를 맺어준다.

  

![[KakaoTalk_20250323_080348923.jpg]]

  

1. 애플리케이션 컨텍스트는 @Bean 어노테이션이 붙은 메소드의 이름으로 빈 목록을 만든다.
2. 클라이언트의 요청이 들어오면 애플리케이션 컨텍스트는 빈 목록에서 해당 빈을 찾는다.
3. 빈이 있다면 해당 클래스에 빈 생성을 요청한다.
4. 빈 생성 후 클라이언트가 해당 빈을 사용할 수 있게 만들어준다.

  

**이처럼 애플리케이션 컨텍스트는 모든 빈을 관리하면서 객체간 관계를 설정해주는 역할을 한다.**

  

### **스프링 IOC 용어 정리**

  

**빈(Bean) :** 스프링이 IOC 방식으로 관리하는 오브젝트, 스프링을 사용하는 애플리케이션의 모든 오브젝트가 빈은 아니다. **스프링이 직접 생성과 제어를 담당하는 오브젝트만 빈이다.**

  

**빈 팩토리(Bean Factory) :** 빈을 등록, 생성, 조회, 리턴해주는 IOC 핵심 컨테이너

보통 빈 팩토리를 바로 사용하지 않고 애플리케이션 컨텍스트를 활용

  

**애플리케이션 컨텍스트(Application Context) :** 빈 팩토리를 확장한 IOC 컨테이너

기본적인 기능은 빈 팩토리와 동일하지만 스프링이 제공하는 각종 부가 서비스를 추가 제공한다.

**빈 팩토리는 빈의 생성과 제어의 관점이고 애플리케이션 컨텍스트는 스프링이 제공하는 애플리케이션 지원 기능을 모두 포함하는 것**

**ApplicationContext는 BeanFactory를 상속한다.**

  

**설정정보 / 설정 메타정보(Configuration metadata) :** 어플리케이션 컨텍스트 또는 빈 팩토리가 IOC를 적용하기 위해 사용하는 메타정보

  

**컨테이너 또는 IOC 컨테이너**

애플리케이션 컨텍스트 = 컨테이너 or 스프링 컨테이너

빈 팩토리 = IOC 컨테이너
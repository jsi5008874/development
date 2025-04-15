
### 생성자 주입의 작동 방식

```java
@Component // MyService를 빈으로 등록 
public class MyService { 
	public void doSomething() { System.out.println("서비스 로직 실행"); } 
}

@Controller // MyController를 빈으로 등록 
public class MyController { private final MyService myService; 

public MyController(MyService myService) { // 생성자 주입 
this.myService = myService; }

public void execute() { myService.doSomething(); } 
}
```

**생성자 주입 방식에서 Spring이 빈을 주입하는 과정**
1️.  Spring이 `MyController`를 빈으로 생성해야 함
2️.  `MyController`는 `MyService`를 생성자 파라미터로 받음
3️.  Spring 컨테이너는 `MyService` 타입의 빈을 찾아서 자동으로 주입
4️.  `MyService`가 이미 빈으로 등록되어 있다면, 기존 빈을 주입  
5️.  만약 `MyService`가 아직 빈으로 등록되지 않았다면, Spring이 먼저 `MyService`를 생성한 후 `MyController`에 주입

**생성자 주입 방식은 애플리케이션 컨텍스트에서 빈을 찾을 때 타입기반으로 빈을 찾아서 반환해준다.**
만약 같은 타입의 빈이 존재한다면 @Primary 또는 @Qualifier를 활용해서 특정 빈을 명시해서 찾아와야한다.

**정리하자면 MyController가 생성 될 때 Spring이 자동으로 MyService 빈을 찾아서 주입**
**즉 개발자가 new MyService()를 호출할 필요 없이 Spring이 자동으로 관리하고 주입해준다.**

### @Autowired 주입(필드 주입)의 작동 방식

```java
@Component 
public class MyController { 
@Autowired 
private MyService myService;
}
```

**필드 주입 방식에서 Spring이 빈을 주입하는 과정**
1. MyController 객체 생성(의존성 없는 상태)
2. @Autowired 필드 주입 실행(리플렉션 사용)

**자바 리플렉션을 활용하여 런타임 시 강제로 주입을 하는 방법이다.**


### 필드 주입(@Autowired)이 안좋은 이유

1. 필드 주입은 final 키워드를 사용할 수 없어서 객체의 불변성을 보장하지 못한다.
		final은 객체가 최초에 생성될 때의 값을 그대로 유지하는 것인데 필드 주입은 객체를 생성한 후
		필드에 주입을 하는 방식이기 때문에 final 키워드가 쓰이면 불가능하다.
2. 필드 주입은 순환 참조 문제가 발생
		생성자 주입은 순환 참조를 컴파일 시점에 감지 가능하지만 필드 주입은 런타임시에 주입을 하기 때문에
		미리 감지하지 못하고 순환 참조 발생 가능성이 있다.
3. 필드 주입은 테스트에 용이하지 못하다.
		스프링 컨텍스트가 있으면 @Autowired 자동 주입 가능하지만 속도가 느리다.
		또한 스프링 컨텍스트 없이 단일 테스트 하려면 필드 주입 방식은 리플렉션으로 처리해야해서 코드도 복잡해지고 방식이 불편하다.

### JdbcContext 분리
현재까지의 전략 패턴의 구조를 보면
UserDao의 메소드 = 클라이언트
익명 내부 클래스 = 전략 (바뀌는 부분)
jdbcContextWithStatementStrategy() = 컨텍스트(바뀌지 않는 부분)

jdbcContextWithStatementStrategy() 메소드는 jdbc의 일반적인 작업 흐름을 가지고 있기 때문에 다른 Dao에서도
활용이 가능하다.
그래서 jdbcContextWithStatementStrategy()를 UserDao 밖으로 독립시켜 모든 Dao가 사용할 수 있도록 만든다.

### 클래스 분리
JdbcContext라는 클래스를 생성하고 jdbcContextWithStatementStrategy()를 옮긴다.
workWithStatementStrategy()로 이름을 바꿔서 JdbcContext 클래스에 넣는다.

```java
public class JdbcContext{
	private DataSource dataSource;

	public void setDataSource(DataSource dataSource){
		this.dataSource = dataSource;
	}

	public void workWithStatementStarategy(StatemebtStrategy stmt) throws SQLExeption{
		Connection c = null;
		PreparedStatement ps = null;

		try{
			c = this.dataSource.getConnection();
			ps = stmt.makePreparedStatement(c);
			ps.excuteUpdate;
		}
		catch(SQLExeption e){
			throw e;
		}..........
	}
}
```
DB 커넥션을 필요로 하는 코드는 JdbcContext 클래스로 옮겼기 때문에 DataSource 타입의 빈을 받을 수 있게 작성한다.

**UserDao에서 JdbcContext 활용하기**
```java
public class UserDao{
	......
	private JdbcContext JdbcContext;

	public void setJdbcContext(JdbcContext jdbcContext){
		this.jdbcContext = jdbcContext;
	}

	public void add(final User user) throws SQLExeption{
		this.jdbcContext.workWithStatementStrategy(new StatementStrategy){
			.....
		}
	}
}
```
UserDao의 외부에서 jdbcContext 클래스를 DI 받아 활용한다.
이렇게 활용하면  UserDao외 다른 Dao에서도 활용이 가능하다.


### 스프링 빈으로 DI

![[KakaoTalk_Photo_2025-04-15-23-06-28.jpeg]]
현재 구조를 보자면 인터페이스를 사용하지 않고 클래스 레벨에서 구체적인 의존관계가 만들어진다.

인터페이스를 사용하지 않고 DI를 적용하는 것은 문제가 없을까?
꼭 DI를 인터페이스를 통해서 하지 않아도 상관 없다.

현재와 같은 상황은 온전한 DI라고 볼 수 없지만 스프링의 DI는 넓게 보자면 객체의 생성과 관계설정에 대한 제어권한을
오브젝트에서 제거하고 외부로 위임했다는 IOC의 개념을 포괄한다.
**(UserDao가 객체 생성 및 관계 설정을 jdbcContext에게 위임)**

그런 의미에서 DI의 기본을 따르고 있다.

**JdbcContext를 UserDao와 DI 구조로 만들어야하는 이유**
1. JdbcContext가 싱글톤 레지스트리에서 관리되는 싱글톤 빈이 되기 때문이다.
		JdbcContext는 그 자체로 변경되는 상태정보를 가지고 있지 않다.
		내부에서 사용할 DataSource라는 인스턴스 변수는 있지만 DataSource는 읽기 전용이므로 JdbcContext가 싱글톤이 되는데
		아무 문제 없다.
		JdbcContext는 JDBC 컨텍스트 메소드를 제공해주는 일종의 서비스 오브젝트로서 의미가 있고 그래서 싱글톤으로 등록돼서
		여러 오브젝트에서 공유해 상용되는 것이 이상적이다.

2. JdbcContext가 DI를 통해 다른 빈에 의존하고 있기 때문이다.
		JdbcContext는 DataSource 프로퍼티를 통해 DataSource 오브젝트를 주입받도록 되어 있다.
		DI를 위해서 주입되는 오브젝트와 주입받는 오브젝트 모두 스프링 빈으로 등록되어야 스프링의 IOC 대상이 된다.
		DI를 받기 위해서는 빈으로 등록되어야한다.


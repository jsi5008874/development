
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


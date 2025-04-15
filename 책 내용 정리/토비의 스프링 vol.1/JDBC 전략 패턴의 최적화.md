전략 패턴으로 클라이언트, 컨텍스트, 전략인터페이스, 전략 클래스로 구분하면서
중복되는 코드를 많이 줄였지만 여전히 안좋은점이 있다.

**우선 DAO 메소드마다 새로운 StatementStrategy 클래스를 만들어야하고 User 같은 부가적인 정보가 있으면**
**오브젝트를 전달받는 생성자와 이를 저장해둘 인스턴스 변수를 만들어야한다.**

이를 해결하면 더 깔끔한 구조가 완성 될 것이다.


### 로컬 클래스
클래스 파일이 많아지는 문제점을 해결하기 위해 중첩 클래스(nested class)를 활용하면 된다.
중첩 클래스는 다른 클래스 내부에 정의되는 클래스를 말한다.
중첩 클래스도 나뉘는데 독립적으로 오브젝트로 만들어질 수 있는 스태틱 클래스,
자신이 정의된 클래스의 오브젝트 안에서만 만들어질 수 있는 내부 클래스로 나뉜다.

내부 클래스는 다시 범위(scope)에 따라 세 가지로 분류된다.
멤버 필드처럼 오브젝트 레벨에 정의되는 멤버 내부 클래스(member inner class),
메소드 레벨에 정의되는 로컬 클래스(local class), 이름을 갖지 않는 익명 내부 클래스(anonymous inner class)이다.

로컬 클래스를 활용해 StatementStrategy 전략 클래스를 UserDao의 메소드 내부에 넣는다.

```java
public void add(User user) throws SQLExeption{
	class AddStatement implements StatementStrategy{
		User user;
		public AddStatement(User user){
			this.user = user;
		}
		.......//전략 클래스 코드
	}

	//클라이언트 코드
	StatementStrategy st = new AddStatement(user);
	jdbcContextWithStatementStrategy(st);
}
```

이렇게 전략 클래스 코드를 클라이언트의 메소드 내부에 넣어서 클래스 파일을 최소화한다.

### 익명 내부 클래스
AddStatement 클래스는 add() 메소드에서만 사용하는 로컬 클래스이다.
좀 더 간결하기 위해 이름도 제거하는 익명 내부 클래스를 활용할 수 있다.

```java
 //익명 내부 클래스는 구현하는 인터페이스를 생성자처럼 이용해서 오브젝트 생성
public void add(final User user) throws SQLExeption{
	jdbcContextWithStatementStrategy(
		new StatementStrategy(){
	public PreparedStatement makePreparedStatement(Connection c) throws SQLExeption{
		PreparedStatement ps = c.prepareStatement("insert into user(id, name, password) values(?,?,?)")
		ps.setString(1, user.getId());
		ps.setString(2, user.name());
		ps.setString(3, user.password());
		return ps;
		}
}
	);
}
```
이렇게 jdbcContextWithStatementStrategy의 파라미터에서 바로 생성해서 활용


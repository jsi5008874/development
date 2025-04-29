
3장에서 jdbcContext로 만들었던 코드를 jdbcTemplate을 적용하도록 바꾸면서 SQLExeption이 사라졌다.

```java
public void deleteAll() throws SQLExeption{
	this.jdbcContext.excuteSql("delete from users");
}

-------------------------------------------------------------------------

public void deleteAll(){
	this.jdbcTemplate.update("delete from users);
}
```

무엇때문에 SQLExeption이 사라졌고 사라져도 아무 이상이 없는 이유는 무엇일까?

[[Java 에러와 예외]]를 먼저 보고오자


## 예외처리 방법

### 예외 복구
예외상황을 파악하고 문제를 해결해서 정상 상태로 돌려놓는 것
ex) 사용자가 요청한 파일을 읽으려고 시도했는데 해당 파일이 없거나 다른 문제가 있어서 읽히지 않아 IOException 발생

이때는 사용자에게 상황을 알려주고 다른 파일을 이용하도록 안내해서 예외상활을 해결
예외로 인해 기본 작업 흐름이 불가능하면 다른 작업 흐름으로 자연스럽게 유도해주는 것

이는 기능적으로는 사용자에게 예외상황으로 비쳐도 애플리케이션에서는 정상적으로 설계된 흐름을 따라 진행된다.

### 예외처리 회피
예외처리를 자신이 담당하지 않고 자신을 호출한 쪽으로 던져버리는 것
throws를 선언해서 예외가 발생하면 알아서 던져지게 하거나 catch문으로 일단 예외를 잡은 후 로그를 남기고 다시 예외를 던지는 것

```java
public void add() throws SQLExeption{
	try{
		//코드
	}
	catch(SQLExeption e){
		throw e;
	}
}
```
이렇게 throws를 통해서 자신을 호출한 쪽으로 예외를 던져준다.

하지만 서로 긴밀한 관계가 아닌데 무작정 예외를 던지면 좋지 않다.
템플릿/콜백 패턴에서 콜백에서 예외가 발생해서 템플릿으로 예외를 던져주는 상황은 괜찮지만
별로 밀접하지 않은 객체간의 관계에서 갑자기 예외를 던지면 무책임한 책임회피가 된다.

따라서 자신을 사용하는 쪽에서 예외를 다루는 게 최선의 방법이라는 분명한 확신이 있는 상태에서 예외처리 회피를 해야한다.

### 예외 전환
예외 회피와 비슷하지만 발생한 예외를 그대로 넘기는 것이 아니라 적절한 예외로 전환해서 던진다.

ex) 새로운 사용자를 등록하려고 시도했을 때 동일한 아이디가 등록되어 있어 DB 에러가 발생하면 SQLException을 그대로 보내는게 아니라
DuplicateUserIdException 같이 명확한 예외로 바꿔서 던져주는 것이다.

```java
public void add(User user) throws DuplicateUserIdException, SQLExeption{
	try{
		//코드
	}
	catch(SQLExeption e){
		//ErrorCode가 MySQL의 "Duplicate Entry(1062)"이면 예외 전환
		if(e.getErrorCode()==MysqlErrorNumbers.ER_DUP_ENTRY)
			throw DuplicateUserIdException();
		else
				throw e; //그 외의 경우는 그대로 SQLExeption	
	}
}
```
이렇게 예외를 전환해서 던져주면 훨씬 빨리 어떤 상황인지 파악하여 조치를 취할 수 있다.

보통 전환을 할 때는 중첩 예외로 만드는 것이 좋다.
중첩 예외로 만들어서 처음 발생한 예외가 무엇인지 확인할 수 있게 만드는 것이다.
initCause() 메소드 활용
```java
catch(SQLExeption e){
	....
	throw DuplicateUserIdException().initCause(e);
}
```
이렇게 SQLExeption의 정보도 담아서 던져주면 처음 발생한 예외도 확인할 수 있다.

**또 다른 전환 방법은 포장하는 것이다.**
포장은 보통 예외 처리를 강제하는 체크 예외를 언체크 예외인 런타임 예외로 바꾸는데 사용한다.

대표적인 예시는 EJBException이 있다.
EJB 컴포넌트 코드에서 발생하는 대부분의 체크 예외는 비즈니스 로직으로 볼 때 의미 있는 예외이거나 복구 가능한 예외가 아니다.
이런 경우에는 런타임 예외인 EJBException으로 포장해서 던지는게 낫다.
```java
try{
	//코드
} catch(NamingException ne){
	throw new EJBException(ne);
} catch(SQLException se){
	throw new EJBException(se);
}
```
이렇게 런타임 예외로 만들어서 던지면 EJB 컴포넌트를 사용하는 다른 EJB나 클라이언트에서 일일이 예외를 잡거나 던지는 수고를 할 필요 없다.

어차피 복구가 불가능한 예외라면 가능한 빨리 런타임 예외로 포장해 던지게 해서 다른 계층의 메소드를 작성할 때 불필요한 throws를
선언하지 않도록 해줘야한다.

대부분 서버 환경에서는 애플리케이션 코드에서 처리하지 않고 전달된 예외들을 일괄적으로 다룰 수 있는 기능을 제공한다.
복구가 안되는 예외는 예외처리 서비스 등을 이용해 자세한 로그를 남기고 관리자에게 알림으로 알리고 사용자에게는
친절한 안내 메시지를 보여주는 식으로 처리하는게 바람직하다.


## 예외처리 전략

### 런타임 예외의 보편화
자바 엔터프라이즈 서버 환경은 수많은 사용자가 동시에 요청을 보내고 각 요청이 독립적인 작업으로 취급 받는다.
예전의 독립형 애플리케이션과 달리 서버의 특정 계층에서 예외가 발생했을 때 사용자와 바로 커뮤니케이션 하면서 예외 상황을
복구할 방법이 없기 때문에 하나의 요청을 처리하는 중에 예외가 발생하면 해당 작업만 중단시키면 그만이다.

차라리 애플리케이션 차원에서 예외상황을 미리 파악하고 예외가 발생하지 않도록 차단하는게 좋다.
그냥 요청의 작업을 취소하고 서버 관리자나 개발자에게 통보해주는 편이 훨씬 낫다.

따라서 대응이 불가능한 체크 예외라면 빨리 런타임 예외로 전환해서 던지는게 훨씬 낫다.
그러면 throws Exception으로 점철된 의미 없는 메소드들이 줄어들 것이다.

그래서 점점 런타임 예외가 보편화 되고 있다.

### add() 메소드의 예외처리
예외 전환 첫 번째 예시에 DuplicateUserIdException과 SQLException 두 가지 체크 예외를 던진 add() 메소드가 있다.

DuplicateUserIdException은 충분히 복구 가능한 예외이므로 add() 메소드를 사용하는 쪽에서 잡아서 대응할 수 있다.
하지만 SQLExeption은 대부분 복구 불가능한 예외이므로 잡아봤자 처리할 것도 없고 throws를 타고 계속 앞으로 전달되다가
어플리케이션 밖으로 던져질 것이다. 그럴바엔 그냥 런타임 예외로 포장해 던져버려서 다른 메소드들이 신경쓰지 않게 해주는게 좋다.

DuplicateUserIdException은 굉장히 의미 있는 예외이므로 add() 메소드를 호출한 오브젝트 말고 더 앞단의 오브젝트에서도
다룰 수 있다. 어디에서든 처리할 수 있다면 굳이 체크 예외로 만들지 않고 런타임 예외로 만드는게 낫다.

```java
public class DuplicateUserIdException extends RuntimeException{
	public DuplicateUserIdException(Throwable cause){
	super(cause);
	}
}
```
이렇게 RuntimeException을 상속한 런타임 예외로 만들고 중첩 예외를 만들 수 있도록 생성자도 만들었다.

이를 통해 add() 메소드를 다시 변경해보면

```java
public void add() throws DuplicateUserIDException{
	try{
		//코드
	}
	catch(SQLExeption e){
		if(e.getErrorCode()==MysqlErrorNumbers.EP_DUP_ENTRY)
			throw new DuplicateUserIdException(e); //예외 전환
		else
			new RuntimeException(e); //예외 포장
	}
}
```
이렇게 하면 add() 메소드를 사용하는 오브젝트는SQLExeption을 처리하기 위해 불필요한 throws 선언을 할 필요가 없으며
필요한 경우 아이디 중복 상황을 처리하기 위해 DuplicateUserIdException을 사용할 수 있다.


### 애플리케이션 예외
시스템 또는 외부의 예외상황이 아니라 애플리케이션 자체의 로직에 의해 의도적으로 발생시키고
반드시 catch해서 무엇인가 조치를 취하도록 요구하는 예외가 있는데 이를 애플리케이션 예외라 한다.

예를 들어 사용자가 요청한 금액을 은행계좌에서 출금하는 기능을 가진 메소드가 있다고 가정
무턱대고 출금을 허용하면 안되고 현재 잔고를 확인하고 허용하는 범위를 넘어서 출금을 요청하는 작업을 중단하고
적절한 경고를 사용자에게 보내야하는 상황이 있다.

이런 기능을 담은 메소드를 설계하는 방법이 두 가지 있다.

첫 번째는  정상적인 출금처리를 했을 경우와 잔고 부족이 발생했을 경우에 각각 다른 종류의 리턴 값을 돌려준다.
정상 출금이 되면 요청 금액 자체를 리턴해주고 실패하면 0이나 -1같은 특별한 값을 리턴해준다.
이 메소드를 호출한 쪽에서는 리턴 값을 확인하여 분기처리를 해줘야한다.
이 방법은 예외상황에 대한 명확한 표준이 있어야한다.
어떤 개발자는 0을 리턴하고 누구는 -1을 누구는 -999 같은 숫자를 리턴할 수 도 있기 때문에 예외상황에 대한 일관적인 정책이 있어야한다.

두 번째는 정상적인 흐름을 따르는 코드는 그대로 두고 예외 상황에서는 비즈니스적인 의미를 띤 예외를 던지도록 만드는 것이다.
잔고 부족인 경우 InsufficientBalanceException 등을 던진다.
정상적인 흐름을 따르지만 예외가 발생할 수 있는 코드를 try 블록 안에 정리해두고 예외상황에 대한 처리는 catch 블록에 모아 둘 수 있다.
이 때 사용하는 예외는 체크 예외로 만든다. 개발자가 잊지 않고 잔고 부족처럼 자주 발생 가능한 예외상황에 대한
로직을 구현하도록 강제해주는 것이다.

### SQLException은 어떻게 됐나?
먼저 SQLException이 복구 가능한 예외인지 생각해봐야한다.
대부분은 복구가 불가능하다.
통제할 수 없는 외부 상황이거나 프로그램의 오류 또는 개발자의 부주의로 생기는 오류이다.
SQL 문법이 틀리거나 DB 서버가 다운됐거나 DB 커넥션 풀이 꽉 차서 DB 커넥션을 가져올 수 없는 상황들이 있다.
이는 애플리케이션 레벨에서 복구가 불가능하고 빨리 개발자에게 예외 발생 사실을 알리는 편이 낫다.

따라서 예외처리 전략을 적용해서 가능한 빨리 언체크/런차임 예외로 전환해줘야 한다.

jdbcTemplate 템플릿과 콜백 안에서 발생하는 모든 SQLExeption은 런타임 예외인 DataAccessException으로 포장해서 던진다.
런타임 예외이기 때문에 처리가 꼭 필요한 경우에는 잡아서 처리를 하고 그 외에는 무시해도 된다.

이렇게 하면 쓸데없는 throws SQLExeption이 전파되는 걸 막을 수 있고 위 의 예시에서 본 것 처럼 SQLException이 사라진것이다.
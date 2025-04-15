  

### Dirty Read

커밋되지 않은 변화를 읽음

예제)

![[image 93.png]]

최초에 DB에는 X=10, Y=20이라는 값이 있었는데

트랜잭션 1번 > x+y, 트랜잭션 2번 y를 70으로 바꾸는 두 개의 트랜잭션을 실행

이 때 1번 트랜잭션 실행 도중 2번 트랜잭션이 실행되면

원래 1번 트랜잭션의 결과는 30이어야 하지만 도중에 y가 70으로 바뀌면서 결과가 80이 됨

그리고 1번 트랜잭션 커밋 후 2번 트랜잭션을 롤백하게 되면?

isolation 관점에서 문제가 생기는 것임

이처럼 커밋하지 않은 값을 읽어서 처리하는 것을 dirty read라고 한다,

  

### Non-repeatable read

![[image 94.png]]

위의 예시처럼 같은 데이터를 두번 읽었는데 서로 값이 다름

Fuzzy read라고 불리기도 한다.

  

### Phantom read

![[image 95.png]]

갑자기 없던 데이터가 생겨서 isolation 관점에서는 이상한 부분이 생긴것임

  

  

**위의 세 가지 이상현상들을 모두 발생하지 않게 만들 수 있지만 그러면 제약사항이 많아져서**

**동시 처리 가능한 트랜잭션 수가 줄어들어 DB 처리량이 저하된다.**

  

**일부 이상한 현상은 허용하는 몇 가지 level을 만들어서 개발자가 필요에 따라**

**적절하게 선택할 수 있도록 하는것이 isolation level이다.**

  

![[image 96.png]]

여기서 serializable은 세 가지 이상현상 뿐만 아니라 모든 이상현상이 일어나지 않는 level을 의미

  

위의 isolation level은 ANSI에서 표준 SQL 92를 발표하면서 나온 개념인데

허술한 부분이 있어서 이에 반박하면서 나온 개념들이 아래와 같다.

![[image 97.png]]

  

### Dirty write

![[image 98.png]]

최초에 x=0이라는 데이터가 있다고 가정

둘 다 write를 하는 두 개의 트랜잭션이 있는데 여기서 1번이 x=10으로 write 하고

2번이 x=100으로 write 한 후 1번이 rollback을 하면 x는 1번이 write하기 전인 x=0으로 바뀌게 된다.

이처럼 commit이 되지 않은 데이터를 write해버리면 rollback을 할 때 정상적인 recovery가 안되기 때문에 이런 dirty write는 모든 isolation level에서 허용하면 안된다는 주장이 생겼다.

  

### Lost update

![[image 99.png]]

  

### 확장된 dirty read

![[image 100.png]]

롤백이 되지 않아도 dirty read가 될 수 있음

최초 x=50, y=50일 때

이체를 하더라도 두 개의 합이 100이어야하는데

트랜잭션 2번이 중간에 수행됐을 때는 60이 되버림

### Read skew

![[image 101.png]]

데이터의 일관성이 없음

  

### write skew

![[image 102.png]]

x+y≥0은 테이블의 제약사항을 표현한 것으로 한쪽이 마이너스가 되더라도 두 개의 합이

0보다 크거나 같기만하면 상관없다는 뜻

최초의 값은 x=50, y=50이라고 가정

위와 같이 트랜잭션을 실행하면 두 합이 -70이 되면서 제약사항을 위반하게 되고

일관성이 없는 데이터를 가지게 된다.

  

### 확장된 Phantom read

![[image 103.png]]

cnt는 v의 값이 10 이상인 튜플의 개수를 세는 것

위의 트랜잭션들을 보면 꼭 두 번 read하지 않더라도 중간에 순서가 잘못되면

의도하지 않은 값이 read 될 수 있다.

  

**이처럼 기존의 SQL 표준 isolation level을 비판하며 나온것이 snapshot isolation level이다.**

### Snapshot isolation level

snapshot : 특정 시점의 형상

여기서는 트랜잭션 시작점을 특정 시점으로 정하고 트랜잭션 시작점일 때의 데이터 상태를 의미

  

![[image 104.png]]

트랜잭션 1번이 시작되는 시점인 x=50, y=50을 스냅샷에 가지고 있다가 문제가 생기면

해당 데이터로 롤백을 하게되는 것임

또한 write를 했다고 해서 DB에 적용하는 것이 아니고 스냅샷에 해당 데이터를 가지고 있다가

commit 시 DB에 적용

  

![[image 105.png]]

트랜잭션 1번의 read(y)를 보면 50으로 읽는데 그 이유가 위에서 설명한 것처럼

현재 DB의 값을 읽는게 아니라 해당 트랜잭션이 시작한 시점의 스냅샷을 가지고 있다가

스냅샷의 값을 읽어오기 때문에 50을 읽는 것

또한 위의 예시를 보면 y에 대한 write가 두 개인데 이처럼 write conflict가 생기면

먼저 커밋한 트랜잭션만 살리고 나머지는 rollback을 시킨다.

  

  

### 실무에서 isolation level 적용

  

mysql

![[image 106.png]]

SQL 표준 isolation level 사용

  

Oracle

![[image 107.png]]

SQL 표준을 사용하지만 Read uncommited는 사용하지 않고

Reapeatable read, Serializable 두 개는 동일하게 Serializable을 적용해서

결국 Read committed, Serializable 두 개만 사용

  

Sql server

![[image 108.png]]

snapshot 까지 포함해서 5개 사용

  

Postgresql

![[image 109.png]]

Reapeatable read가 snapshot의 기능을 가짐



**출처 : 유튜브 쉬운코드 https://www.youtube.com/@ezcd
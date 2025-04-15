  

### Unrecoverable Schedule

![[image 270.png]]

위의 예시에서 빨간색 트랜잭션이 롤백이 된다면 초록색 틀랜잭션의 read는 잘못된 데이터를

읽어서 트랜잭션을 수행하게되는 꼴이된다.

정상적인 경우라면 초록색 트랜잭션은 빨간색 트랜잭션이 롤백되었기 때문에 220만원이 최종결과가 되어야하지만 이미 초록색 트랜잭션이 커밋 된 후에 빨간색 트랜잭션이 롤백되었기 때문에

문제가 생긴다.

이러한 경우를 Unrecoverable Schedule이라고 한다.

**Unrecoverable Schedule은 롤백을 해도 이전 상태로 회복되는것이 불가능할 수 있기 때문에**

**DBMS가 허용하면 안된다.**

  

### Recoverable Schedule

![[image 271.png]]

즉 초록색 트랜잭션의 read(h_balace)가 바로 위의 파란색 트랜잭션 write(h_balance)를 의존한다.

파란색 트랜잭션이 write한 값, 즉 트랜잭션으로 인해 변동이 있는 데이터를 초록색 트랜잭션이

read를 하고 있기 때문에 의존성이 있는것이다.

이 때 의존을 받고있는 파란색 트랜잭션이 먼저 커밋 또는 롤백을 하고 의존을 하고있는 초록색 트랜잭션이 나중에 커밋 또는 롤백을 하면 결과 값이 바뀌지 않게된다.

이러한 개념이 Recoverable Schedule이다.

  

### Cascading rollback

![[image 272.png]]

의존성에 따라 연쇄적으로 롤백하는것을 의미한다.

하지만 여러 트랜잭션의 롤백이 연쇄적으로 발생하면 비용이 높아진다.

  

연쇄적인 롤백을 방지하기 위해 나타난개념이 cascadeless schedule이다.

  

### cascadeless schedule

![[image 273.png]]

이렇게 의존을 받는 트랜잭션을 먼저 커밋 또는 롤백을 하고 의존하는 트랜잭션을 실행시키는 방법

  

스케줄 내에서 어떤 트랜잭션도 커밋 되지 않은 트랜잭션들이 write한 데이터를 읽지 않는 경우

하지만 cascadeless schedule도 완벽하진 않다.

그래서 더 엄격하게 생긴 개념이 Strict Schedule이다.

  

### Strict Schedule

예시) H사장님이 3만원이던 피자 가격을 2만원으로 낮추려는데

K직원도 동일한 피자의 가격을 실수로 1만원으로 낮추려 했을 때

![[image 274.png]]

위의 예시를 보면 사장님이 2만원으로 write를 하고 커밋했지만 직원이 1만원으로 한 뒤 롤백을 하면

결국 피자 가격은 3만원이 된다.

이유는 두 트랜잭션이 동일한 데이터에 접근을 하는데 먼저 직원이 피자 가격이라는 데이터에 접근해서 write를 하고 사장님이 이후에 직원이 변경시킨 데이터에 접근해서 2만원을 write해서 커밋했지만 먼저 접근했던 직원이 rollback을 하면 최초에 직원이 접근했던 데이터 값인 3만원으로 돌아가기 때문에 문제가 발생한다.

그래서 cascadeless schedule보다 엄격하게 commit되지 않은 트랜잭션들이 write한 데이터는 쓰지도 읽지도 않는 경우의 Strict Schedule을 적용한다.

  

![[image 275.png]]

최종적으로 이런 모습이 되어야한다.

  

![[image 276.png]]

recoverable schedule은 롤백으로 원래의 상태로 돌아갈 수 있는 스케줄이고

거기서 cascading rollback이 발생하지 않도록 하는 것이 cascadeless schedule이고

그보다 더 엄격하게 만든 것이 strict schedule이다.

  

  

![[image 277.png]]

결국 Concurrency control은 serializability와 recoverability를 만족하는것이고

이는 트랜잭션의 Isolation 고립성과 관련이 있는 개념들이다.



**출처 : 유튜브 쉬운코드 https://www.youtube.com/@ezcd
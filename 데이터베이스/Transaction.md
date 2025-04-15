  

단일한 논리적인 작업단위

논리적인 이유로 여러 SQL문들을 단일 작업으로 묶어서 나눠질 수 없게 만든 것

transaction의 SQL문들 중에 일부만 성공해서 DB에 반영되지 않음

  

![[image 72.png]]

  

### Java에서 트랜잭션 예제

  

![[image 73.png]]

직접 DB 커넥션 및 auto commit 설정을 해줘야함

  

### Spring에서 트랜잭션 예제

  

![[image 74.png]]

@Transaction 어노테이션을 활용하면 위의 java 예제코드에서 봤던 connection과 관련된 코드들을

없애고 필요한 로직만 작성하면 됨

  

### ACID

A : atomicity(원자성)

C : consistency(일관성)

I : isolation(고립성)

D : durability(영존성)

  

### Atomicity(원자성)

![[image 75.png]]

완전히 성공하거나 실패하거나 둘 중 하나

개발자는 언제 commit 하거나 rollback 할지를 챙겨야한다.

  

### consistency(일관성)

![[image 76.png]]

일관성의 예시) 80만원 있는 계좌에서 100만원을 다른 계좌로 이체한다면 이는 일관성에 위배된다.

계좌의 잔액은 0보다 크거나 같아야하는데 -20만원이 되기 때문에 DB에서 정의한 룰에 위배되기 때문이다.(DB설계 시 잔액 컬럼은 0보다 크거나 같다고 정의 했다는 가정)

  

이처럼 Transaction을 수행할 때 DBMS에서 일관성을 확인하고 commit 또는 rollback을 한다.

  

### isolation(고립성)

ex) J가 H에게 20만원을 이체할 때 H도 ATM에서 본인 계좌의 30만원을 입금한다면

![[image 77.png]]

두 개의 트랜잭션이 겹쳐서 30만원이 사라지게 됨

처음 J가 보낸 20만원의 트랜잭션은 처음 read 했을 때 200만원이기 때문에 30만원 트랜잭션을 완료해도 20만원 트랜잭션이 완수될 때는 220만원을 write하게됨

  

![[image 78.png]]

여러 isolation level로 나누는 이유는 엄격한 고립성으로 통제하면 병렬처리가 어려워져 DB의 성능이 떨어지게되서 개발자가 필요에 의해 정할 수 있게 만든 것임

  

### durability(영존성)

![[image 79.png]]



**출처 : 유튜브 쉬운코드 https://www.youtube.com/@ezcd
  

DB에서의 Lock은 OS에서의 Lock과 거의 같다고 보면된다.

  

예제

![[image 110.png]]

트랜잭션 1번이 먼저 write_lock을 취득

트랜잭션 2번이 read_lock을 취득해서 read하려고 해도 1번이 먼저 write_lock을 취득했으므로

read를 하지 못하고 대기

1번이 write를 하고 unlock으로 lock을 반환하면 2번이 lock을 획득해서 read를 할 수 있음

  

이처럼 OS에서 봤던 lock 개념과 거의 비슷함

  

하지만 write lock과 read lock의 차이점을 알아야함

  

### write lock

![[image 111.png]]

exclusive(배제적인) lock으로 wrtie lock이 실행중일 때 다른 트랜잭션이 read/write 둘 다

못하게 막아주는 것

  

### read lock

![[image 112.png]]

다른 트랜잭션이 read 하는 것은 허용함

lock을 거는 이유가 write 또는 read를 할 때 해당 데이터에 변형이 일어나는 것을

방지하기 위함인데 다른 트랜잭션이 read를 하더라도 해당 트랜잭션이 read하는데

아무런 지장이 없기 때문이다.

  

![[image 113.png]]

  

  

### lock을 써도 생기는 이상한 현상 예제

![[image 114.png]]

최초 x=100, y=200

해당 예제의 경우 serial schedule은 그림 왼쪽과 같이 두 개이다.(t1 > t2, t2 >t1)

하지만 예제처럼 트랜잭션이 진행될 경우에는 x=300, y=300으로 serial schedule 두 개 경우의 수와

일치하지 않는 Nonserializable인 상황이다.

lock을 사용했는데도 이런 상황이 발생하는 이유는?

  

t2의 입장에서 결국 바뀌는 값인 y를 write 하기전에 t1에서 y에 대해 read_lock을 걸었기 때문이다.

즉 t2가 y를 사용하기전에 t1이 y에 접근해서 문제가 생긴 것이다.

  

![[image 115.png]]

이를 해결하기 위해 t2에서 unlock(x)를 하기전에 y에 대한 lock을 취득해줘야한다.

여기서 문제가 생겼던 이유는 결국 serializable하게 작동하려면 트랜잭션 2번이 계산을 한 후

y에 바뀐 데이터를 삽입하고 트랜잭션 1번이 작동을 해야하는데

y를 업데이트 하기 전에 트랜잭션 1번이 y값을 읽어서 그렇다.

따라서 t2가 write_lock(y)를 먼저 취득하고 unlock(x)를 해줘야함

  

이러한 개념은 2PL 프로토콜이라 한다.

  

### 2PL protocol

![[image 116.png]]

t2를 기준으로 설명하면 트랜잭션에서 모든 locking operation(read_lock(x), write_lock(y))이

최초의 unlock operation보다 먼저 수행되는 것을 뜻하고

two phase의 뜻은 lock을 취득만 하는 Expanding phase와 lock을 반환만 하는 Shrinking phse

두 개의 페이즈로 나눠져 있다는 것을 뜻함

  

2PL protocol은 serializability를 보장한다,

  

### 2PL과 Deadlock

![[image 117.png]]

t2가 read_lock(x)를 취득하고 t1이 read_lock(y)를 먼저 취득 한 상황

여기서 2PL이기 때문에 t1이 unlock(y)를 하기전에 write_lock(x)를 취득하려고 하지만 t2가 가지고 있어서 대기

t2도 마찬가지로 t1이 y에 대한 lock을 가지고 있어서 대기

이렇게 데드락이 발생

데드락 해결방법은 OS 데드락 해결방법과 비슷함

  

### Conservative 2PL

![[image 118.png]]

모든 lock을 먼저 취득하고 트랜잭션을 시작하는데 모든 lock을 취득하는게 어려울 수 있음

따라서 실용적이진 않다.

  

### Strict 2PL

![[image 119.png]]

strict schedule(recoverability, 즉 롤백을 했을 때 원래의 데이터를 보장하는 개념)를 보장하는 2PL

commit 또는 rollback 후 write-lock을 반환한다.

  

### Strong Strict 2PL

![[image 120.png]]

write 뿐만 아니라 read-lock도 commit/rollback 이후에 반환

구현은 더 쉽지만 read-lock의 반환 시점이 더 늦어진다는 단점이 있다.

  

  

  

read-read의 경우를 제외하면 한쪽이 block이 되는데 이러면 전체 처리량이 좋지않아서

read-write가 서로 block하는것을 해결하기위해 나온것이

## MVCC(Multiversion concurrency control)이다.

![[image 121.png]]

  

[[MVCC(Multiversion Concurrency Control)]]



**출처 : 유튜브 쉬운코드 https://www.youtube.com/@ezcd
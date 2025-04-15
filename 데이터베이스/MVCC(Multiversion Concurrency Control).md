  

기본적으로 MVCC는 commit 된 데이터만 읽는다.

  

여기부터는 lock, unlock을 그림에 따로 표시하지않음

예를 들어 write(x)를 했으면 write_lock을 취득한 것으로 알고

commit이 됐다면 unlock을 한 것으로 이해하면 된다.

  

### 예제

![[image 278.png]]

최초의 X=10이었음

  

t1의 첫번 째 read는 x의 최초값인 10으로 읽지만 현재 isolation level이 read commited이므로

t2가 write하고 commit한 이후에 읽어온 두번 째 read는 50으로 읽는다.

  

![[image 279.png]]

하지만 isolation level이 repeatable read라면 트랜잭션이 시작된 시간 기준으로 그 전에 commit된

데이터를 읽어오기 때문에 t2가 x=50으로 write 하더라도 t1이 시작되기 전 시점인 x=10으로 읽음

  

![[image 280.png]]

여기서 MVCC의 특징을 보면

1. 데이터를 읽을 때 **특정 시점 기준**으로 가장 최근에 commit된 데이터를 읽는다

>> 위에서 살펴본 read commited, repeatable read와 같이 isolation level에 따라 특정 시점의 기준이 달라진다.(mysql에서는 이러한 특징을 consistent read라고 부른다.)

1. 데이터 변화(write) 이력을 관리한다.

>> 데이터 변화에 대한 히스토리를 저장해서 관리하기 때문에 MVCC는 RDBMS의 저장장소를 차지한다.

1. read와 write는 서로를 block하지 않는다.

>> 위에서도 봤듯이 read와 write가 서로를 block하지 않아서 락에 비해 성능이 더 좋다.

  

### PostgreSQL의 lost update 해결

  

최초 값 x=10, y=50

![[image 281.png]]

두 트랜잭션이 serializable하게 동작한다면 x= 40, y=50이 되어야한다.

하지만 보다시피 두 트랜잭션의 isolation level이 read commited일 때 값은 x=80, y=50이 나왔다.

즉 t1의 업데이트 내용이 사라진 것이다.(lost update)

  

![[image 282.png]]

그래서 t2의 isolation level을 repeatable read로 올렸다.

postgreSQL에서 repeatable read의 특징이 있는데 그건 바로

같은 데이터에 먼저 update한 트랜잭션이 commit되면 같은 데이터에 나중에 접근한 트랜잭션은

롤백한다는 규칙이 있다.

  

이 때문에 t2의 write는 롤백이 된다.

롤백이 된후 다시 트랜잭션 2번이 실행된다면 결과는 serializable한 값 즉 x=40, y=50으로 나온다.

  

![[image 283.png]]

t1과 t2의 isolation level은 그대로 두고 순서만 바꿔서 t2를 먼저 실행 >> t1 실행으로 했다.

아까와 동일한 트랜잭션, 동일한 isolation level로 진행 했지만 똑같이 lost update가 발생했다.

(t2 입금한 부분이 사라짐)

  

![[image 284.png]]

그래서 t1도 reapeatable read로 바꿔서 실행하면 정상적으로 작동한다.

  

**예제에서 봤듯이 lost update를 해결하려면 한 쪽의 트랜잭션 isolation level을 신경쓰는게 아니고**

**연관되어 있는 다른 트랜잭션들의 isolation level도 동일하게 변경해주어야 한다.**

  

**하지만 mysql의 repeatable read에는 나중 트랜잭션을 rollback하는 규칙이 없어서**

**repeatable read만으로는 lost update를 해결할 수 없다.**

  

### MySql에서 lost update 해결

  

Locking read의 개념

![[image 285.png]]

mysql에는 locking read라는 개념이 있는데

write를 하기전에 우선 해당 데이터에 어떤 데이터가 있는지 read를 먼저 한 후에 write를 한다.

따라서 쿼리문을 생성할 때 뒤에 For Update라는 구문을 넣으면 read를 하더라도 자동으로

write-lock을 취득하게 된다.

이런 개념이 locking read이다.

  

![[image 286.png]]

여기서 locking read의 특징으로는 가장 최근의 commit된 데이터를 읽는다는 점이다.

현재 t1은 repeatable read로 원래는 t1이 시작한 시점의 x를 읽어서 x=50으로 읽어야하지만

locking read로 lock을 취득했기 때문에 가장 최근에 commit 된 80을 읽게된다.

  

![[image 287.png]]

최종적으로 x=40, y=50으로 serializable한 결과를 얻게된다.

mysql에서는 repeatable read만으로는 제대로된 결과를 얻기 힘들고 locking read를 해줘야한다.

  

locking read에는 For Update(write-lock), For Share(read-lock) 두 가지가 있다.

  

  

### mysql write skew 해결

![[image 288.png]]

serializable하게 작동한다면 위와 같은 결과가 나와야한다.

  

![[image 289.png]]

하지만 repeatable read라면 위와 같이 적용된다.

  

![[image 290.png]]

하지만 repeatable read라도 locking read를 사용하면 정상적인 값이 나옴

  

  

### postgreSql write skew 해결

  

postgresql에도 locking read가 있는데 동작방식이 조금 다르다.

  

![[image 291.png]]

mysql과 동일하게 locking read를 취득해서 동작한다해도

postgre의 repeatable read 특성 때문에 결국 rollback이 됨

  

  

### MySql의 serializable 레벨

![[image 292.png]]

기본적으로 모든 트랜잭션의 평범한 select문에 암묵적으로 for share가 붙음

즉 자동으로 read-lock을 취득하는 select문으로 생성됨



**출처 : 유튜브 쉬운코드 https://www.youtube.com/@ezcd
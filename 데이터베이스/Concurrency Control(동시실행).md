  

### Schedule

여러 트랜잭션들이 동시에 실행될 때 각 트랜잭션에 속한 operation들의 실행순서

  

operation이란?

![[image 80.png]]

위의 예제에서 트랜잭션이 실행될 때 실행되는 행동의 단위를 operation이라고 한다.

  

![[image 81.png]]

operation들은 위의 그림처럼 간략하게 표현할 수 있다.

read는 r로 write는 w로 표현하고 commit은 c로 표현

r1, w1처럼 숫자는 1번 트랜잭션이라는 뜻이고 (k), (H)는 operation에 들어갈 인자를 뜻함

  

![[image 82.png]]

이처럼 두 개의 트랜잭션이 실행된다고 가정하면 많은 경우의 수가 생기는데

이 때 operation의 실행순서를 Schedule이라 한다.

하지만 케이스가 달라지더라도 각 트랜잭션 내의 operation들의 순서가 바뀌지는 않는다.

ex) case1에서도 트랜잭션1번의 순서는 r1(K) >> w1(K) >> r1(H) >> w1(H)인데

case2, 3, 4에서도 트랜잭션 내의 순서는 바뀌지 않고 똑같다.

  

### Serial Schedule

![[image 83.png]]

위 그림처럼 트랜잭션 1번 완료 > 트랜잭션 2번 수행 또는 트랜잭션 2번 완료 > 트랜잭션 1번 수행

이렇게 트랜잭션들이 겹치지 않고 한번에 하나씩 수행되는것을 Serial Schedule이라한다.

  

### Non Serial Schedule

![[image 84.png]]

  

### Serial, NonSerial Schedule의 성능

![[image 85.png]]

NonSerial Schedule의 성능이 Serial Schedule의 성능보다 더 좋다.

그 이유는 Serial Schedule은 한 트랜잭션이 read, write의 I/O작업을 하는 동안 CPU가 놀게되지만

Non Serial Schdule은 트랜잭션1이 I/O작업을 하는동안 트랜잭션2를 CPU에서 동시에 실행하기 때문

  

### Non Serial Schedule의 단점

![[image 86.png]]

case 4의 예시처럼 최종적으로 H가 250만원을 가져야하는데 트랜잭션이 겹쳐서 220만원이 되는것 처럼 이상한 결과가 나올 수 있다.

  

  

  

![[image 87.png]]

Serial이 안전하긴 하지만 Non Serial이 빠르기 때문에 Serial과 동일한 Non Serial을 실행한다는 생각을 하게 되었고 Schedule이 동일하다의 의미를 정의하기 시작함

  

### Conflict

![[image 88.png]]

Conflict는 서로 다른 트랜잭션 소속의 operation이 같은 데이터에 접근하면서 최소 하나는 write

위의 예시를 보면 w2(h)와 r1(h)는 같은 데이터에 접근하면서 서로 다른 트랜잭션 소속이고

w2가 write이기 때문에 read-write conflict가 된다.

  

![[image 89.png]]

이렇게 write-write conflict도 있음

  

conflict가 중요한 이유는 conflict operation의 순서가 바뀌면 결과도 바뀌기 때문이다.

![[image 90.png]]

해당 예시처럼 w2를(H가 자기 계좌에 30만원 입금) 하고나서 J가 H에게 입금하기위해

H의 계좌를 조회하는 r1(h)를 하면 r1(h)의 값은 230이지만

순서가 바뀌어서 r1(h)를 먼저 하면 H가 자기 계좌에 입금하기 전의 시점이기 때문에

r1(h)가 200으로 바뀌게 된다.

  

### Conflict Equivalent

![[image 91.png]]

스케줄 2, 3이 있을 때 두 개의 스케줄은 서로 같은 트랜잭션을 가졌다.(조건 1번 만족)

  

조건 2번의 뜻은 스케줄 3은 3개의 conflict가 있음

r2(h)와w1(h), w2(h)와r1(h), w2(h)와w1(h) 총 3개의 conflict가 있는데

해당 conflict의 순서가 스케줄 2에서도 동일해야한다.

예시를 보면 스케줄 3의 w2(h), r1(h)의 conflict와 같이 스케줄 2의 w2(h), r1(h)도

w2(h)가 먼저 실행되고 r1(h)가 실행된다.

이렇게 두 스케줄의 모든 conflict의 순서가 같아야 conflict equvalent라고 한다.

  

### Conflict Serializable

![[image 92.png]]

여기서 중요한 점은 스케줄 2는 serial schedule이라는 점이다.

즉 serial schedule과 conflict equivalent일 때 Conflict Serializable이라고 하고

Conflict Serializable이면 Non serial Schedule이더라도 결과가 바뀌지 않는다.

  

  

실제로 구현할 때는 여러 트랜잭션이 실행될 때마다 해당 스케줄이 Conflict Serializable인지

확인을 하면 너무 오래걸리기 때문에

**여러 트랜잭션을 동시에 실행해도 스케줄이 Conflict Serializable하도록 보장하는 프로토콜을 적용**

  

[[Recoverability]]



**출처 : 유튜브 쉬운코드 https://www.youtube.com/@ezcd
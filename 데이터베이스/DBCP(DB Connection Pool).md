  

![[image 199.png]]

Backend Server와 DB Server는 TCP 기반으로 커넥션을 유지한다.

여기서 문제는 서로 통신을 할 때마다 Connection을 열고 닫고 해야하는데

이게 시간적인 비용이 많이 발생하게 된다.

TCP는 3 way handshake로 open connection을 하고 4 way handshake로 close connection을 한다.

즉 한 번 커넥션을 열고 닫는데 통신을 7번 주고 받아야하는 리소스가 생기는 것이다.

  

  

![[image 200.png]]

이런 단점을 보완하기 위해 DBCP 개념이 생겼다.

DBCP는 마치 스레드풀처럼 계속 열려있는 커넥션을 여러개 생성해서 pool에 넣어놓는다.

그리고 커넥션을 사용해야할 때 get connection으로 커넥션을 가져와서 DB 서버와 통신을 하고

통신이 끝나면 close connection으로 커넥션을 DBCP로 반납한다.

>> 여기서 close connection은 커넥션을 닫는게 아니라 DBCP로 반납하는 개념이다.

  

이렇게 커넥션을 재사용하면서 열고 닫는 시간을 절약해서 서비스의 성능을 향상 시킨다.

  

  

### DBCP 설정방법(mysql, Hikari CP 기준)

  

DB 서버 설정

![[image 201.png]]

max_connections : client와 맺을 수 있는 최대 connection 수

  

그림을 예시로 들면 위에 있는 백엔드 서버 혼자서 운영을 하다가 서비스가 커지면서

서버에 과부하가 생겼고 똑같은 서버를 한대 더 추가해서 부하를 분산하는 상황

하지만 현재 DB 서버의 max_connections가 4이고 기존의 백엔드 서버도 DBCP에 connection을 4로

유지해서 DB 서버가 client와 맺을 수 있는 모든 connection이 채워져 있다.

이 때 새로 백엔드 서버를 추가해도 이미 DB 서버는 max_connections를 모두 소진해서

새로운 백엔드 서버와 connection을 맺을 수 없게 된다..

  

  

![[image 202.png]]

wait_timeout : connection이 inactive(놀고있는중) 중에 다시 요청이 오기까지 대기할 시간

  

그림을 예시로 보면 현재 커넥션이 연결되어 있는 상태인데

비정상적으로 connection이 종료되거나 connection을 다 쓰고 반환이 안되거나 네트워크 단절 등

비정상적인 상황으로 요청이 오지 않는다면?

DB에서는 계속 기다리기만 하게 될 것이다.

이런 상황을 방지하기 위해 wait_timeout을 설정해서 설정한 시간만큼 기다려도 요청이 오지 않는다면 그냥 close 해버린다.

  

  

백엔드 서버 설정

![[image 203.png]]

minimumIdle : pool에서 유지하는 최소한의 idle(놀고있는, 대기중인) connection 수

maximumPoolSize : pool이 가질 수 있는 최대 connection 수(대기 중인 애, 일하는 중인 애 합친 값)

  

여기서 minimumIdle = 2, maximumPoolSize = 4일 때 생기는 일

최초에는 대기 중인 커넥션 2개만 유지를 하다가 요청이 들어와서 한 커넥션이 일을 하게되면

idle connection = 1, active connection = 1 이렇게 된다..

하지만 minimumIdle = 2 이기 때문에 바로 커넥션풀에 idle connection을 1개 추가해서

idle connection = 2,, active connection = 1 의 상황으로 바꿔준다.

이렇게 요청이 계속 들어오면 idle connection을 minimumIdle의 크기만큼 계속 보충을 해주지만

maximumPoolSize에 넘지 않는 숫자 만큼만 보충을 해준다.

현재 예시에서는 4를 넘지 않게 한다.

  

![[image 204.png]]

하지만 HicariCP 공식문서의 권장사항은 minimumIdle의 기본 값은 maximumPoolSize와

동일하게 설정하라고 한다.

이유는 만약 트래픽이 집중되는 타이밍에 connection을 새로 생성해서 connection pool에

새로 배치하는 리소스가 생기기 때문에 처음부터 maximumPoolSize와 동일하게 설정해서

커넥션을 최대한 많이 보유하고 있어야한다.

  

  

![[image 205.png]]

maxLifeTime : pool에서 connection의 최대 수명

  

DB의 wait_timeout보다 몇 초 짧게 설정해야한다.

>> DB의 wait_timeout, Hicari CP의 maxLifeTime 모두 60초라고 가정

이 때 DBCP의 커넥션 중 하나가 maxLifeTime이 59초 일 때 active로 활성화 되었고

60초에 DB 서버에게 요청을 보냈고 가는 중간쯤에 DB의 커넥션은 wait_time인 60초가 되어서

연결을 끊어버린다.

이렇게 되면 요청을 보냈지만 이미 DB에서 해당 커넥션을 없앴기 때문에 exception이 발생

이런 상황을 방지하기 위해 Hicari CP의 maxLifeTime을 더 짧게 지정해서 요청을 보내는 중간에

DB에서 커넥션을 close하는 상황을 방지한다.

  

  

![[image 206.png]]

connectionTimeOut : pool에서 connection을 받기 위한 대기 시간

  

만약 CP에 모든 커넥션이 active 상태라면 해당 요청은 커넥션들이 idle 상태로 올 때 까지

대기해야한다. 이런 최대 대기 시간을 설정하는 파라미터이다.

  

  

### 적절한 connection 수를 찾기 위한 과정

  

![[image 207.png]]

1. 우선 모니터링 환경을 구축
2. nGrinder 같은 백엔드 시스템 부하 테스트를 실시
3. request per second(단위초 당 요청 처리), avg response time(평균 응답 시간)을 측정
4. 이 때 RPS, ART 모두 일정하다가 어느 순간 급변 하는 시점이 생긴다.
5. 이렇게 급변하는 시점에 모니터링 툴을 통해서 서버의 각종 지표를 분석

  

![[image 208.png]]

1. 백엔드 서버와 DB 서버의 CPU, memry 등 리소스 사용률을 확인
2. 백엔드 서버의 과부하 인지 DB 서버의 과부하 인지 판단
3. 백엔드 서버 과부하라면 백엔드 서버를 증설, DB 서버 부하면 slave DB 증설, 샤딩 등 적용
4. 그리고 백엔드 서버에서는 thread per request 모델이라면 현재 active thread 수를 확인한다.
5. 현재 스레드 풀의 maxSize가 10인데 active thread도 10 이라면 스레드 풀에서 병목현상 발생
6. 이 때 스레드 풀의 maxSize를 늘려준다.
7. 스레드 풀의 문제가 아니라면 DBCP의 active connection 수 확인
8. active connection이 maximumPoolSize와 같다면 DBCP의 커넥션이 부족한 것
9. 이 때는 DB서버의 max_connections를 조금씩 늘려보면서 부하테스를 진행해서

적절한 DB connection의 수를 찾아서 설정한다..



**출처 : 유튜브 쉬운코드 https://www.youtube.com/@ezcd
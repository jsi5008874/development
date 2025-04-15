  

정의 : 두 개 이상의 프로세스 혹은 스레드가 서로가 가진 리소스를 기다리는 상태

리소스 : CPU, 임계영역, 모니터 등등등 프로그램이 돌아가는데 필요한 것들

  

### 데드락을 만드는 네가지 조건

1. Mutual exclusion : 리소스를 공유해서 사용할 수 없다.
2. Hold and wait : 프로세스가 하나 이상의 리소스를 취득한(hold) 상태에서 다른 프로세스가

사용하고 있는 리소스를 추가로 기다린다(wait)

1. No preemption : 리소스 반환(release)은 오직 그 리소스를 취득한 프로세스만 할 수 있다.
2. Circular wait : 프로세스들이 순환(circular) 형태로 서로의 리소스를 기다린다.

>> P1은 P2가 가진 리소스를 P2는 P3가 가진 리소스를 P3는 P1이 가진 리소스를 기다림

  

### OS의 데드락 해결 방법

1. 데드락 방지(Deadlock prevention)

>> 데드락의 네 가지 조건 중 하나가 충족되지 않게 시스템을 디자인

가. 리소스를 공유 가능하게함(현실적으로 불가능 ex. 프린트는 한번에 하나만 출력)

나. 사용할 리소스들을 모두 획득한 뒤 시작 / 리소스를 전혀 가지지 않은 상태에서만 리소스

요청

다. 추가적인 리소스를 기다려야 한다면 이미 획득한 리소스를 다른 프로세스가 선점 가능하

도록 한다.

라. 모든 리소스에 순서 체계를 부여해서 오름차순으로 리소스를 요청(가장 많이 사용됨)

  

1. 데드락 회피(Deadlock avoidance)

>> 실행 환경에서 추가적인 정보를 활용해서 데드락이 발생할 것 같은 상황을 회피하는 것

Banker algorithm : 리소스 요청을 허락해줬을 때 데드락이 발생할 가능성이 있으면

리소스를 할당해도 안전할 때 까지 계속 요청을 거절하는 알고리즘

  

1. 데드락 감지와 복구

>> 데드락을 허용하고 데드락이 발생하면 복구하는 전략

가. 프로세스를 종료시킨다.(리스크가 큼)

나. 리소스의 일시적인 선점을 허용한다.(다른 프로세스가 가진 리소스를 할당 받음)

  

1. 데드락 무시

>> OS가 아무것도 안함, 개발자가 알아서 풀어야됨 ㅋㅋㅋ

  

### java 데드락 예제

  

public class Main{

……… main(String[] args){

Object lock1 = new Object();

Object lock2 = new Object();

  

Thread t1 = new Thread(() → {….});

Thread t2 = new Thread(() → {….});

  

t1.start();

t2.start();}}

  

Thread t1 = new Thread(() → {

synchronized(lock1){ //t1 스레드가 lock1을 취득 후 임계영역 진입

System.out.println(”[t1] get lock1”); //t1이 lock1을 취득

synchronized(lock2){ //t1이 lock1 취득 후 lock2를 취득해서 새로움 임계영역 진입  
System.out.println(”[t1] get lock2”);}}} //t1이 lock2를 취득  

);

  

Thread t2 = new Thread(() → {  
synchronized(lock2){ //t2 스레드가 lock2을 취득 후 임계영역 진입  
System.out.println(”[t2] get lock2”); //t2이 lock2을 취득  
synchronized(lock1){ //t2이 lock1 취득 후 lock1를 취득해서 새로움 임계영역 진입  
System.out.println(”[t2] get lock1”);}}} //t2이 lock1를 취득  
);  

  

위 코드에서 만약 t1이 lock1을 취득했을 때 동시에 t2가 lock2를 취득한다면 교착상태 발생

  

여기서 해결을 하려면 여러 방법이 있지만

1. t1 또는 t2의 lock 순서를 바꿈 ex) t2의 lock2 취득 후 lock1을 lock1 취득 후 lock2 취득으로 바꾸게되면 데드락이 발생하지 않음
2. 지금 코드 상 두 스레드 모두 lock을 중첩해서 획득하게 되어있음(lock1 취득 후 새로운 임계영역에 들어가기 위해 lock2를 획득하는 것) 이런 중첩이 꼭 필요한가? 생각해보고 바꿀 수 있다면 중첩을 없애서 데드락을 방지


**출처 : 유튜브 쉬운코드 https://www.youtube.com/@ezcd
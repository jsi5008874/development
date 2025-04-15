  

mutual exclusion(상호배제)을 보장

조건에 따라 스레드가 대기(waiting) 상태로 전환 가능

  

모니터는 언제 사용되나?

1. 한번에 하나의 스레드만 실행돼야 할 때
2. 여러 스레드와 협업(cooperation)이 필요할 때

  

모니터의 구성요소

1. mutex : critical section에서 mutual exclusion을 보장하는 장치

critical section에 진입하려면 mutex lock을 취득해야 함

mutex lock을 취득하지 못한 스레드는 큐에 들어간 후 대기(waiting)상태로 전환

mutex lock을 쥔 스레드가 lock을 반환하면 락을 기다리며 큐에 대기 상태로 있던

스레드 중 하나가 실행

  

1. condition variable : waiting queue를 가짐, 조건이 충족되길 기다리는 스레드들이 대기상태로

머무는 곳

condition variable에서 주요 동작(operation)

wait : 스레드가 자기 자신을 condition variable의 waiting queue에 넣고 대기 상태로 전환

signal : waiting queue에서 대기중인 스레드 중 하나를 깨움

broadcast : waiting queue에 대기중인 모든 스레드를 깨움

  

모니터의 뼈대가 되는 코드

  

acquire(m); //모니터의 락 취득 mutual exclusion을 보장하기 위해 락을 mutex가 가져감

while(!p){ //조건확인

wait(m, cv);} //조건 충족 안되면 waiting

….. 이런저런 코드들…….

signal(cv2); 또는 boradcast(cv2); //cv2가 cv와 같을 수도 있음

release(m); //모니터의 락 반환 실행 끝난 후 mutex가 락 반환

  

### Mutex의 역할

1. T1이 락을 취득해서 임계영역에 들어간 상태에서 T2가 임계영역에 접근한다면 이미 락은 T1이 가지고 있으므로 mutex는 mutual exclusion을 보장하기 위해 T2를 acquire()단계에서 못들어가게 막고 큐에 넣는다.(entry queue)
2. T1이 작업을 끝내고 release()를 하면 mutex는 T2를 entry queue에서 꺼내서 임계영역에 들어가 실행하게 해준다.

즉, mutex는 entry queue를 관리하면서 임계영역의 mutual exclusion을 보장

  

### condition variable의 역할

1. 어떤 스레드가 임계영역에 들어갔을 때 조건이 충족되지 않으면(위의 코드 상 while문) wait를 호출한다.
2. 이 때 wait에 m(mutex lock)과 cv(condition variavle)을 인자로 가지고 오는데 mutex lock을 가지고 오는 이유는 현재 임계영역에 도달했지만 조건을 충족하지 못해 대기 상태에 들어가야하는 스레드가 mutex lock을 계속 가지고 있으면 그 동안 다른 스레드들이 임계영역에 들어오지 못하기 때문에 mutex lock을 반환해주는것이고 condition variable을 호출해서 자신은 waiting queue에 들어가서 대기하게 됨
3. 이후에 조건이 충족되서 waiting queue에서 나온 후 이런 저런 코드들을 실행 한 후 signal 혹은 broadcast를 실행하는데 이 때 들어가는 인자는 상황에 따라 다르다.

cv가 될 수도 있고 cv 뒤에 waiting queue에서 기다리고 있던 cv2(두번째로 잠든 스레드)일 수도 있다.

1. 실행이 완료되면 release를 통해 락 반환

즉, condition variable은 임계영역 안에서 waiting queue를 관리하며 임계영역 내부를 스케쥴링 해줌

  

### entry queue : critical section에 진입을 기다리는 큐(mutex가 관리)

### waiting queue : 조건이 충족되길 기다리는 큐(condition variable이 관리)

  

### bounded producer / consumer problem

>> 아이템을 계속 생산해서 고정된 크기의 버퍼에 계속 넘겨주는 프로듀서가 있고

버퍼로 부터 아이템을 컴슘해서 필요한 처리를 하는 커슈머가 있다.

여기서 버퍼가 가득 차있을 때 프로듀서는 버퍼에 빈공간이 있는지 끊임없이 확인해야하는가?

그리고 버퍼가 비어있는 상태라면 컨슈머는 계속 버퍼에 아이템이 있는지 확인해야하는가?

하는것이 bounded producer / consumer problem이다.

이 문제를 모니터를 통해 해결이 가능하다.

  

ex)

  

global Buffer buffer;

global Lock lock;

global CV fullCV;

global CV emptyCV;

  

public method producer(){

while(true){

task myTask = ………..; //producer의 task를 정의

lock.acquire(); //락을 취득

  

while(buffer.isFull()){ //버퍼가 가득 차 있는지 확인

wait(lock, fullCV);} //가득 차 있으면 락을 반환하고 waiting queue에 들어감

  

buffer.enqueue(myTask); //task를 수행

signal(emptyCV); 또는 broadcast(emptyCV); //emptyCV를 깨움

  

lock.release();}} // 락 반환

  

public method consumer(){  
while(true){  
lock.acquire(); // 락 취득  

  
while(buffer.isempty()){ // 버퍼가 비어있는지 확인  
wait(lock, emptyCV);} // 비어있다면 lock을 반환하고 waiting queue에 들어감  

  

myTask = buffer.dequeue(); // 버퍼에서 아이템을 꺼내옴

  
signal(fullCV); 또는 broadcast(fullCV); // fullCV를 깨워줌  

  
lock.release();}} // 락 반환  

  

doStuff(myTask); // task 수행

  

  

프로듀서 한개를 P1, 컨슈머 한개를 C1으로 가정(entry queue, waitiong queue, buffer 모두 비워진 상황)

  

1. C1이 먼저 락을 취득하여 임계영역에 도달했지만 현재 버퍼는 비워져 있는 상태이므로 waiting queue에 들어감(락 반환)
2. P1이 락을 취득하여 임계영역에 도달 > 현재 버퍼가 비워져 있으므로 P1은 task 수행해서 buffer를 채워줌
3. signal을 통해 waiting queue에 있던 C1을 깨워줌
4. 이 때 C1은 다시 임계영역에 들어가려하지만 현재 P1이 락을 가지고 있는 상태이므로 mutex에 의해 entry queue에서 대기
5. P1은 signal 끝낸 후 락을 반환
6. entry queue에 있던 C1은 락을 취득 후 임계영역 도달
7. 현재 버퍼는 P1이 수행한 task1이 들어가 있는 상태이므로 C1이 task1을 가져옴
8. C1은 signal을 통해 fullCV를 깨워줌
9. 하지만 현재 waiting queue는 비워져 있으므로 아무런 일도 일어나지 않음
10. C1은 락을 반환 후 C1의 task를 수행

  

이런식으로 producer와 consumer는 서로의 task를 수행하면 서로 깨워주고 대기하며 효율적으로 task를 수행한다.

  

### 자바에서 모니터란

자바에서는 모든 객체는 내부적으로 모니터를 가진다.

  

모니터의 mutual exclusion 기능은 synchronized 키워드로 사용한다.

자바의 모니터는 condition variable을 하나만 가진다.

  

자바 모니터의 세가지 동작

-wait

-notify(위 예제의 signal과 같음)

-notifyAll(위 예제의 broadcast와 같음)

  

자바 모니터를 사용할 때 두 가지 이상의 condition variable이 필요하다면

따로 구현이 필요함

  

java.util.concurrent에는 동기화 기능이 탑재된 여러 클래스들이 있으니 참고


**출처 : 유튜브 쉬운코드 https://www.youtube.com/@ezcd
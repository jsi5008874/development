  

![[OS_프로세스_상태변화 1.png]]

  

1. 프로세스가 생성되면 new 상태로 대기하고 롱텀 스케줄러가 허가하면 ready 상태로 변한다.

>> 하지만 대부분은 바로 ready 상태로 진입(대부분 OS에는 롱텀 스케줄러가 없음)

1. CPU에서 실행 준비단계인 ready 상태에서 스케줄러가 해당 프로세스의 실행차례가 되면 running 상태로 바꿔준다.(scheduler dispatch)
2. running 상태에서 해당 프로세스의 timeslice가 다 되면 다시 ready 상태로 바꾼다.
3. running 단계에서 I/O를 하거나 임계영역에 들어가야하면 wait 상태로 바뀌고 I/O가 끝나거나 임계영역에서의 실행이 끝나면 다시 ready 상태로 들어간다.
4. 종료하는 terminated를 할 때도 ready에서 running을 거쳤다가 teminated 되야한다.

  

다만 운영체제별로 자세한 과정은 다를 수 있다.

  

### JAVA 스레드의 상태 종류

NEW : 자바 스레드가 아직 시작하지 않은 상태

  

RUNNABLE : 실행 중인 상태(다른 리소스를 기다리는 상태 포함)

>> CPU내에서 기다리는 상태도 포함, I/O 작업을 하고 결과를 기다리는 상태도 포함

OS에서의 running 보다 더 포괄적인 상태임

  

BLOCKED : 모니터 락을 얻기 위해 기다리는 상태

>> critical section으로 들어가기 위해 모니터 락을 얻기 위해 기다리는 상태

  

WAITING : 다른 스레드를 기다리는 상태

>> Object.wait, Thread.join 등등

  

TIMED_WAITING : 제한 시간을 두고 다른 스레드를 기다리는 상태

>> Object.wait with timeout, Thead.join with timeout, Thread.sleep 등등

  

TERMINATED : 실행을 마치고 종료된 상태

  

### Thread dump : 실행 중인 자바 프로세스의 현재 상태를 담은 스냅샷

프로그램 실행 중 문제가 생겼을 때 thread dump 로그를 확인해서 어느 부분이 문제인지 파악해서 오류를 해결하는 경우가 많음

thread dump 활용법에 대해 잘 알아야함
  

가상 스레드가 I/O bound가 많은 프로그램에 유리한 이유는 I/O 작업이 많으면

스레드의 대기 시간이 길어진다.

따라서 스레드 갯수가 한정적인 OS 스레드를 사용하는 것 보다 JVM 내에서 가상 스레드를 통해 작업을 하다가 CPU 연산이 필요하거나 I/O작업을 하더라도 OS의 도움이 필요한 가상 스레드들만  
JVM에서 스케줄링을 통해 OS 스레드와 매핑해서 CPU연산을 하게 만들고  
그렇지 않은 가상 스레드들은 그대로 JVM에서 관리해서 최대한 많은 양의 요청을 동시에 처리할 수 있도록 해준다.  

  

예시)

가상 스레드 기술 도입 전의 모습은 요청 > OS 스레드와 매핑 > 실행인데

만약 OS에서 실행가능한 스레드가 10개인데 20개의 요청이 들어온다면

나머지 10개의 요청은 OS 스레드로 매핑이 필요하지 않은 작업이어도 스레드풀 대기열에서 대기해야한다.

  

하지만 가상 스레드를 사용하면 JVM에서 대량의 스레드를 생성할 수 있고

OS 스레드가 필요한 시점에만 JVM이 가상 스레드와 OS 스레드를 매핑시켜서 필요한 동작을

실행시킬 수 있다.

  

따라서 일반 스레드 대비 대량의 요청 처리가 가능하다. (I/O bound 기준)

CPU bound는 결국 CPU 연산을 위해 OS 스레드가 필요한 상황이기 때문에

굳이 가상 스레드로 만들어서 JVM에서 스케줄링하고 OS 스레드로 매핑해야하는 번거로움이

필요 없는 것이다.
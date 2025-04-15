  

I/O의 종류

1. network(socket)
2. file
3. pipe : 프로세스간 통신
4. device : 모니터, 키보드 등 외부장치

  

socket : 네트워크 통신은 socket을 통해 데이터가 입출력된다.

백엔드 서버는 네트워크 상의 요청자들과 각각 소켓을 열고 통신한다.

  

### block I/O : I/O작업을 요청한 프로세스/스레드는 요청이 완료될 때까지 블락됨

  

![[block_io.png]]

1. 스레드에서 코드가 실행되다가 read 시스템 콜을 호출하는데 블락킹 시스템 콜이라면 해당 스레드는 블락이 되면서 코드 실행을 멈춤
2. 블락킹 시스템 콜 호출 후 커널에서 read를 실행하고 응답을 보냄
3. 이후에 블락된 스레드는 다시 코드 실행을 시작

  

즉, I/O를 실행하는 동안 스레드가 멈추는 것이 block I/O

  

### soket block I/O

![[socket_block_io.png]]

socket에는 send buffer와 recv buffer(receive buffer)가 있다.

socket S에서 socket A로 데이터를 보낸다고 가정할 때

socket A에서는 받는 입장으로 read 시스템 콜을 보내는데 이 때 데이터가 recv_buffer에 도착할 때 까지 block을 걸고 socket S에서는 write 시스템 콜을 보내는데 send_buffer에 데이터가 가득 차 있다면 버퍼에 빈 공간이 생길 때 까지 블락에 걸린다.

  

### Non-block I/O : 프로세스/스레드를 블락시키지 않고 요청에 대한 현재 상태를 즉시 리턴

  

![[nonblock_io.png]]

1. 스레드가 실행 중 논블락킹 시스템콜 호출
2. 커널에서 read 수행함과 동시에 스레드에 -1 응답(리눅스 기준)을 보냄
3. 스레드는 원하는 데이터는 받지 못했지만 계속 코드를 수행, 커널에서는 아까 요청한 결과 데이터를 가져옴
4. 코드 수행 중 스레드에서 다시 논블락킹 시스템 콜 호출하면 커널에서는 아까 가져온 데이터를 응답해줌

  

즉 스레드가 I/O과정에서도 멈추지않고 다른 일을 수행

  

### socket에서의 non-block I/O

1. 데이터를 받는 socket A는 read 시스템 콜을 받았을 때 recv_buffer에 데이터가 없다면 바로read 시스템 콜을 취소하고 스레드가 하던 일을 계속 하게 함
2. 데이터를 보내는 socket S는 send_buffer가 가득 차 있다면 기다리지 않고 write 시스템 콜을 취소하고 스레드가 하던 일을 하게 함

  

### non-block I/O 결과 처리 방식

1. 완료됐는지 반복적으로 확인

>> 완료된 시간과 완료를 확인한 시간 사이의 갭으로 인해 처리 속도가 느려질 수 있음

1. I/O multiplexing(다중 입출력) : 관심있는 I/O 작업들을 동시에 모니터링하고 그 중에 완료된 I/O작업들을 한번에 알려줌

>> 네트워크 통신에서 많이 사용

1. Callback/Signal 사용

>> 널리 사용되지는 않음

1. io_uring


**출처 : 유튜브 쉬운코드 https://www.youtube.com/@ezcd
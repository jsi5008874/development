  

![[image 233.png]]

1. 어플리케이션이 시스템의 기능을 함부로 쓸 순 없다.

>> 어플리케이션이 네트워크 기능을 사용하고 싶다고 함부로 시스템의 기능(커널 영역의 코드 등)을

사용한다면 현재 PC에 가동중인 다른 프로세스에 문제가 생길 수 있다

따라서 어플리케이션이 시스템의 기능을 함부로 쓸 수 없다는 것이다.

  

1. 어플리케이션이 네트워크 기능을 사용한다면 프로그래밍 인터페이스를 제공한다.

>> 네트워크 인터페이스를 소켓이라고 한다.

  

1. 어플리케이션은 socket을 통해 데이터를 주고 받는다.

>> 어플리케이션은 잘 짜여진 소켓을 통해 다른 프로세스에게 영향이 가지 않도록

데이터를 주고 받는다.

  

1. 개발자는 socket programming을 통해 네트워크 상의 다른 프로세스와 데이터를 주고 받도록

구현한다

>> 개발자가 socket 인터페이스를 구현해서 데이터를 주고 받는다.

  

![[image 234.png]]

하지만 개발을 할 때 소켓을 직접 구현해본적은 없다.

그 이유는 보통 application layer의 프로토콜은 라이브러리나 모듈 형태로 해당 기능이 제공되는데

이미 해당 모듈이나 라이브러리에 소켓을 구현해놨기 때문에 직접 구현할 일은 없다.

  

![[image 235.png]]

실제로 구현된 것을 보면

socket은 <protocol, IP address, port number> 세 가지로 정의된다.

  

![[image 236.png]]

실제 구현 관점에서 socket은 UDP에서는 유니크하게 식별되지만

TCP에서는 유니크하게 식별되지 않는다.

  

그 이유는 TCP의 소켓 동작방식을 알아야한다.

  

![[image 237.png]]

왼쪽이 클라이언트 오른쪽이 서버

Connection 맺는 요청을 기다리는 소켓을 listening socket이라 한다.

  

![[image 238.png]]

커넥션이 맺어지면 listening socket과 맺어지는게 아니라

새로운 소켓을 만들어서 해당 소켓과 커넥션을 이어준다.

  

![[image 239.png]]

여기서 문제는 그럼 서버쪽의 TCP socket은 세 개 모두 IP address, port number가 같은데

어떻게 식별을 할까?

  

![[image 240.png]]

client의 IP address, port number로 식별해서 해당 소켓으로 데이터를 보낸다.

ex) 77.77.77.77에서 온 데이터는 77.77.77.77, 49999로 식별을 해서 위 쪽의 소켓으로 보내고

33.33.33.33에서 온 데이터는 33.33.33.33, 54321로 식별을 해서 아래 쪽의 소켓으로 보낸다.

결국 src IP, src port로 식별을 한다는 뜻이다.

  

**따라서 실제 구현된 <Protocel, IP address, port> 세 가지만으로는 식별이 안되고**

**src IP, src port, dest IP, dest port까지 있어야 식별이 가능하다.**

  

### UDP 소켓 동작 방식

![[image 241.png]]

UDP는 TCP와 다르게 커넥션을 구성하지 않고 전송하는 방식이다.

따라서 IP 위에서 동작하지만 추가적인 기능은 없고 소켓으로 IP address, port만으로 식별해서

데이터를 송수신한다.

  

### Port number 상세 설명

![[image 242.png]]

  

### 프로토콜 표준과 소켓 프로그래밍 차이점 정리

![[image 243.png]]


**출처 : 유튜브 쉬운코드 https://www.youtube.com/@ezcd
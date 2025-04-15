  

![[image 42.png]]

OSI 7 layer는 네트워크 시스템 구성을 위한 범용적인 모델

ISO/IEC에서 OSI 모델을 관리

  

TCP/IP stack은 인터넷과 함께 개발된 프로토콜 스택

IETF에서 인터넷 표준을 관리, TCP, UDP, IP 등의 스펙은 RFC(IETF에서 발행하는 문서)에서 정의한다.

  

![[image 43.png]]

TCP/IP stack을 크게 두 가지로 나눌 수 있는데

  

application layer는 application level에서 네트워크 기능을 사용하는데 목적이 있고

  

아래 세 개의 레이어는 하드웨어/펌웨어, OS 레벨에서 구현하고 네트워크 기능을 지원하는데 목적

>>> System level에서 구현

  

  

### Port의 개념

![[image 44.png]]

프로세스가 시스템과 데이터를 주고 받기 위한 data path 또는 data channel을 뜻한다.

  

application level의 프로세스가 데이터 통신을 하려면 결국 system level 까지 data가 도달해야

데이터 통신이 가능한데 application level의 데이터를 system level과 이어주는 역할을 하는것이

Port이다.

  

### TCP 개발 당시의 데이터 송수신(1970년대)

![[image 45.png]]

당시에는 Internet Protocol(IP)를 사용했는데

IP는 신뢰성이 낮은 프로토콜이었다.

프로세스간 통신 중 데이터 유실이 있을수 있으며, 데이터의 순서를 보장하지도 않았다.

  

하지만 프로세스 간 통신에서 데이터를 안정적으로 주고 받을 수 있는 프로토콜이 필요로 했고

개발된 것이 TCP(Transmission Control Protocol)

**TCP는 IP 위에서 동작한다.**

  

### TCP Connection

![[image 46.png]]

프로세스 간의 안정적이고 논리적인 **통신 통로**를 Connection

1. Connection을 열고(3 way handshake)
2. 데이터를 주고 받고
3. Connection을 닫는다.(4 way handshake)

  

![[image 47.png]]

하지만 port를 어떻게 유니크하게 식별할 것인가?

인터넷을 통해 정보를 주고 받으려면 목적지 프로세스의 포트를 정확히 식별해야 한다.

하지만 포트 number는 65535까지 밖에 표현인 안되서 엄청나게 넓은 인터넷 세상의

모든 프로세스는 65535로 표현할 수 없다.

  

그래서 internet address(IP 주소)와 port number를 합해서 목적지 프로세스의 포트를

유니크하게 식별할 수 있겠다라는 개념이 생겼고

  

### Socket

![[image 48.png]]

그게 바로 소켓이다.

소켓은 internet address + port number로

각 port를 유니크하게 식별하기 위한 **주소**이다.

  

### Connection & Socket

![[image 49.png]]

Connection은 유니크하게 식별 할 수 있어야하고

한 쌍의 socket이 connection을 유니크하게 식별한다.

>> 결국 한 쌍의 socket이 커넥션을 구성하는 것이다.

src socket(source socket) : 요청을 보내는 쪽의 소켓

dest socket(destination socket) : 요청을 받는 쪽의 소켓

  

또한 하나의 소켓은 동시에 여러 커넥션에 사용될 수 있다.

ex) 서버의 프로세스는 동시에 여러 커넥션을 받아야하므로 여러 커넥션에서 사용될 수 있다.

  

### UDP

![[image 50.png]]

커넥션을 맺지 않고 Internet Protocol을 거의 그대로 가져다 써서 신뢰성이 낮은 프로토콜이다

또한 UDP 표준 문서를 보면 socket이라는 단어가 등장하지 않아서 socket을 사용하지 않았는데

  

![[image 51.png]]

이후에 자연스럽게 UDP에서도 socket 개념을 쓰기 시작했다.

  

그래서 Socket은 원래 ip adress + port number였는데

TCP만 사용하다가 UDP도 사용을 하게 되어서

protocol, ip address, port number의 구성으로 바뀌게 되었다.

  

  

하지만 실제 프로그래밍을 할 때는(구현의 관점) 소켓의 개념과 의미가 미묘하게 다르다

특히 소켓을 식별하는 방법에서 큰 차이가 있다.

  

[[표준 프로토콜과 구현 관점의 Socket 차이]]


**출처 : 유튜브 쉬운코드 https://www.youtube.com/@ezcd
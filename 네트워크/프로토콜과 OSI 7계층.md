  

**네트워크 프로토콜 : 네트워크 통신을 하기 위해 통신에 참여하는 주체들이**

**따라야하는 형식, 절차, 규약**

  

![[image 17.png]]

네트워크 통신에도 여러 기능들로 나뉘는데 그림과 같이 어플리케이션에서의 통신

목적지까지 데이터 전송, 노드 사이의 데이터 전송 등 네트워크 통신이 나누어져 있다.

  

여러 기능을 하나의 프로토콜로 구현한다면 유지보수도 어려워지고 복잡해지는 문제가 있어서

여러 기능들을 모듈화하여 설계하였다.

  

![[image 18.png]]

네트워크 통신이 동작하는 것을 하나씩 뜯어보니 마치 각 기능이 계층별로 동작하는 것과 같았다.

  

그림을 예시로 보면 클라이언트 > 서버 통신을 한다고 하면

클라이언트의 어플리케이션 단계에서의 통신 > 라우터 통신 > 서버 어플리케이션 단계의 통신

이렇게 이루어지는데 이는 마치 어플리케이션 계층 > 물리적 계층 > 어플리케이션 계층

이런식으로 계층별 동작을 하는것과 같은 모습이었고

이를 통해 나온 것이 OSI 7계층이다.

  

  

### OSI 7 Layer

![[image 19.png]]

각 레이어에 맞게 프로토콜이 구현되어 있고

각 레이어의 프로토콜은 하위 레이어의 프로토콜이 제공하는 기능을 사용하여 동작

>> network layer는 data link layer의 프로토콜 기능을 사용해서 동작

  

**application layer**

![[image 20.png]]

  

**presentation layer**

![[image 21.png]]

  

**session layer**

![[image 22.png]]

  

  

**application, presentation, session layer가 application lever의 layer**

  

**Transport layer**

![[image 23.png]]

데이터를 전송하는 layer로 목적지까지 어떻게 보낼것인가?를 정의하진 않음

아래의 network layer가 해당 문제에 대한 정의를 하기 때문에 network layer의 기능을 이용하여 구현

  

**network layer**

![[image 24.png]]

데이터를 목적지까지 보내는 기능을 제공

각 라우터에서도 network layer의 기능을 구현해야한다.

하지만 네트워크 통신 중 각 노드들이 어떻게 통신하는지는 정의하지 않고

해당 기능의 정의는 하위 레이어인 data link layer가 정의하고 있음

따라서 network layer도 data link layer의 기능을 가져다 써야한다.

  

**data link layer**

![[image 25.png]]

노드 간의 통신을 담당하는 layer

data link layer는 IP 주소가 아닌 MAC 주소를 기반으로 통신하기 때문에

IP 주소를 MAC 주소로 변환해주는 ARP 프로토콜을 사용

  

예를 들면 data link layer의 통신은 라우터간 통신이기 때문에 IP가 아닌 라우터 장치의

MAC 주소를 식별해서 데이터를 전송하기 때문에 MAC 주소를 기반으로 통신한다.

  

**physical layer**

![[image 26.png]]

실제로 유선 케이블이나 무선 신호 등으로 데이터를 전송하는 계층

  

**요약**

![[image 27.png]]

  

### 예제

![[image 28.png]]

클라이언트에서 서버로 HTTP 요청을 보냈다고 가정

  

1. 클라이언트가 OSI 7 layer를 따라 HTTP 요청을 physical layer까지 포장을 한다.
2. 클라이언트의 physical layer에서 라우터로 전송
3. 라우터의 physical layer에서 데이터를 받으면 network layer까지 포장을 해제하고 안에 있는 데이터를 확인해서 목적지 IP를 확인하고 어디로 데이터를 전송할지 결정
4. 라우터에서 다시 데이터를 포장해서 physical layer로 보냄
5. 서버의 physical layer에서 데이터를 받으면 application layer까지 포장을 해제하면 클라이언트가 보낸 HTTP 요청을 받아서 요청을 처리

  

여기서 포장하는 과정을 encapsulation

포장을 해제하는 과정을 decapsulation


**출처 : 유튜브 쉬운코드 https://www.youtube.com/@ezcd
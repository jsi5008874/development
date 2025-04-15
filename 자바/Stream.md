**Byte 형태로** **데이터를 운반하는데 사용되는 연결통로,** 이는 자료(data)의 흐름이 물의 흐름과 같다는 의미에서 사용  
  
특징 : 물의 흐름과 같이 단방향 통신만 가능하기 때문에 하나의 스트림으로 입/출력을 동시 처리할 수 없다. 또한 먼저 보낸 데이터를 먼저 받게 되어있어서 queue의 FIFO 구조로 되어 있다.  

  

데이터의 근원지를 Source, 데이터의 종착점을 sink, 이것을 연결한것을 Stream이라 한다.

  

inputStream : 입력 받을 때 사용, 하지만 여러 개의 값을 입력해도 한 개의 값만 가져온다.

>> System.in을 사용한다.

outputStream : 출력 할 때 사용, 여러 개의 값을 입력해도 한 개의 값만 가져온다.

>>System.out을 사용한다.

  

**int**

**idata = in.read();** _**// input 은 read 와 연결되어있기 때문에 in.read 를 사욯한다.**_

**out.write(idata);** _**// output 은 write 와 연결되어있기 때문에 out.write 를 사용한다**_

**out.flush();** _**// flush 를 써주지 않으면 출력되지 않는다**_

**out.close();** _**// output 을 끝내는 매서드**_

  

출력을 위해서는 out.write() 이루 flush()와 close() 모두 사용해야한다.

  

한번에 한 값 밖에 못쓴다면 여러 값을 사용하고 싶을때는?

  

### inputStreamReader / outputStreamReader

  

inputStreamReader는 배열을 통해 여러 개의 값을 가져온다. 하지만 내가 쓰는 값이

배열에 정의 된 사이즈보다 크다면 내가 쓴 값을 모두 가져올 수 없고 더 적다면 메모리를 낭비하게된다.

  

이처럼 불편한 점을 해결하기 위해 등장한것이

  

### Buffer

버퍼는 고정값이 아닌 가변값을 받는다.

또한 버퍼는 입력 받은 값을 버퍼에 저장해두고 버퍼가 가득 차거나 개행문자(줄바꿈문자 ex.엔터)가

나타나면 버퍼의 내용을 한 번에 전송하게 된다.

  

버퍼는 기본 타입이 String이기 때문에 int로 계산해야하는 경우 형변환 필요

또한 버퍼는 main 함수 뒤에 throws exception을 사용하거나 try~catch에 넣어줘야 에러가 안난다.
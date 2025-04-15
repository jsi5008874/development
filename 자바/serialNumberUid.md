  

직렬화, 역직렬화와 관련 있는 개념으로

  

![[do-messenger_screenshot_2024-11-20_11_23_10.png]]

이처럼 클래스에 long 형식으로 지정을 한다.

  

용도는 객체의 버전 관리와 인증?의 느낌??

  

들어가기 전에 역직렬화의 과정을 먼저 이해해야한다.

  

### 역직렬화의 과정

![[image 359.png]]

1. 역직렬화의 대상은 클래스의 정보를 가지고 있는데 classpath와 클래스명도 같이 가지고 있어서 해당 classpath를 따라 동일한 class가 있는지 확인한다.
2. 역직렬화 대상이 찾는 클래스가 있다면 해당 클래스의 SerialNumberUid를 비교해본다.

(serialNumberUid가 다르다면 클래스가 변경된 것으로 간주하고 exception 날림)

1. serialNumberUid가 동일하다면 객체로 복원시킨다.

  

  

역직렬화를 할 때 필요한 것이 serialNumberUid(이제부터 SNU로 표현)인데 예시를 통해 보면

customer 클래스에 name 필드만 있었다고 가정

해당 클래스를 직렬화해서 데이터로 사용하다가 다시 역직렬화해서

객체로 사용해야하는 상황인데 역직렬화 전에 갑자기 age 필드가 추가되었다.

이러면 InvalidClassException이 발생을 하게된다.

역직렬화 과정 2번의 내용과 관련이 있는데 개발자가 명시적으로 SNU를 지정하지 않으면

클래스에 변화(필드명 변경, 필드의 데이터 타입 변경 등)가 생기면 자동으로 SNU도 변하기 때문에

역직렬화 해야할 대상의 SNU와 현재 Class의 SNU가 달라서 InvalidClassException이 발생

  

하지만 개발자가 명시적으로 SNU를 지정해 놓으면 class의 내용물이 바뀌더라도 SNU는

변함이 없어서 Exception이 발생하지 않고 역직렬화를 할 수 있다.

  

이 때 유의할 점은 새로 필드를 추가한 경우에는 역직렬화에 성공한다.

age가 새로 추가 됐더라도 객체에는 name=주성, age=null 이런식으로 객체화되는데

  

만약 name 필드의 이름을 naming으로 바꾸게 되면 필드끼리 매핑이 안되서 Exception이 발생하게된다.

>> 역직렬화 해야할 데이터에는 name : 주성으로 되어있지만 현재 클래스의 필드에는 name이 아닌 naming이라는 필드가 존재해서 매핑이 안됨

이런 경우에는 SNU가 동일하더라도 InvalidClassException이 발생

  

![[image 360.png]]

  

이처럼 serialVersionUid는 클래스의 버전관리를 위한 개념으로 이해
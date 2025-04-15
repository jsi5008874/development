  

### **Insertion Anomalies : 삽입 오류**

![[image 122.png]]

이렇게 한 테이블에 모든 컬럼을 넣으면 중복된 데이터가 발생하게 된다.

중복된 데이터는 저장 공간을 낭비하고 실수에 의한 데이터 불일치 가능성이 생김

ex) dept_name 컬럼에 DEV라고 넣어아햐는데 오타로 DEB로 넣는 실수들이 생김

  

부서 배치를 받지 못한 직원이 추가 된다면??

![[image 123.png]]

테이블에 null 값이 많이 생김

  

  

![[image 124.png]]

현재 테이블의 문제점은 별개의 관심사가 한 테이블에 있다는 것

임직원과 부서에 대한 데이터가 한 테이블에 있어서 위에서 말한 문제점들이 생김

  

  

![[image 125.png]]

각 관심사별로 테이블을 나누면 위 와 같이 설계할 수 있고

중복데이터나 null을 최소화할 수 있다.

  

### Deletion Anomalies : 삭제 오류

  

![[image 126.png]]

QA 부서에 마지막 인원이었던 YUJIN이라는 직원의 데이터를 삭제하면

테이블에서 QA 부서에 대한 모든 정보가 사라지게 됨

  

![[image 127.png]]

QA 부서의 정보를 남기기 위해 위처럼 한다면 null도 많고 임직원을 위한 테이블이 부서를 위한 정보만 가지고 있어서 매끄럽지가 않다.

  

### Update Anomalies : 수정 오류

  

최근 DEV 부서 이름이 DEV1로 변경되었다고 가정

![[image 128.png]]

그런데 실수로 JINHO만 DEV1로 변경되었다면 데이터 불일치 발생

  

![[image 129.png]]

하지만 테이블을 나눈다면 데이터 불일치가 생기지 않음

  

### Spurious Tuple(가짜 튜블)

  

![[image 130.png]]

두 테이블은 같은 줄의 데이터들끼리 매칭되어있다.(예시로 사용하기 위해 같은 줄에 있는 데이터가

같은 dept인 것으로 가정)

이렇게 정의 된 두 개의 테이블을 natural join 한다면

  

  

![[image 131.png]]

위처럼 두 개의 가짜 정보가 생긴다.

  

![[image 132.png]]

그 이유는 현재 두 테이블의 natural join은 proj_location의 데이터가 같은 애들끼리 묶어주는데

실제로는 1~4번 줄이 각각 매칭된 총 4개가 나와야하지만

jeju에서 하는 프로젝트가 두 개이기 때문에

1번 줄 : 1001, 2001 Beutiful jeju, jeju, feelm

1번 줄과 3번줄이 join : 1001, 2001, Beutiful jeju, jeju, Picachoo

이런식으로 join에 사용되는 poj_location의 중복으로 인해 가짜 정보가 생성됨

  

  

![[image 133.png]]

이를 방지하기 위해선 위와 같이 3개의 테이블로 나눠서 사용해야한다.

  

  

### null 값이 많이 생기면 발생하는 문제점들

  

![[image 134.png]]

aggregate function : count, sum, max, min 등

  

  

  

![[image 135.png]]

  

**하지만 성능 향상을 위해 테이블을 나누지 않는 경우도 있음**



**출처 : 유튜브 쉬운코드 https://www.youtube.com/@ezcd
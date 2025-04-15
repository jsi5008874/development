  

-사용자가 정의한 프로시저

-RDBMS에 저장되고 사용되는 프로시저

-구체적인 하나의 태스크(task)를 수행한다

  

### 예제1

두 정수의 곱셈 결과를 가져오는 프로시저 작성

  

![[image 58.png]]

프로시저 파라미터를 보면 In과 OUT이 있다.

IN은 평소와 같은 파라미터이고 OUT은 Output parameter로 해당 프로시저의 return 값이라고 생각

  

프로시저 생성 후 call을 통해 프로시저를 실행하고

select로 조회

  

### 예제2

두 정수를 맞바꾸는 프로시저 작성

  

![[image 59.png]]

INOUT은 파라미터로 값을 전달 받을수도 있고 값을 반환해줄 수 도 있음

IN으로 선언하면 해당 파라미터에 다른 값을 넣을 수 없지만 INOUT을 써서 파라미터로 받은 값을

다른 값으로 바꿔 줄 수 있음

ex) IN a 였다면 처음에 swap(5, 4) 이렇게 선언 했을 때 a=5로 계속 고정되고 다른 값으로 바뀔 수 없지만 INOUT으로 선언했기 때문에 a와 b의 값을 바꿀 수 있음

  

![[image 60.png]]

이렇게 결과를 보면 두 값이 바뀌어있음

  

### 예제3

부서별 평균연봉을 가져오는 프로시저 작성

  

![[image 61.png]]

mysql에서 위의 예제처럼 쿼리문이라면 따로 파라미터를 안써줘도 됨

  

### 예제4

사용자가 프로필 닉네임을 바꾸면 이전 닉네임을 로그에 저장하고 새 닉네임으로 업데이트

  

![[image 62.png]]

users 테이블에는 아이디와 현재 닉네임, nickname_logs 테이블에는 예전 닉네임과 바꾼 시간이 있음

  

![[image 63.png]]

바꿀 id와 새로운 닉네임을 파라미터로 전달하면 logs 테이블에 insert, users 테이블에 update하는

프로시저

  

  

### Stored Procedure vs Stored Function

  

![[image 64.png]]

  

  

[[stored procedure를 백엔드 실무에서 쓰기에 조심스러운 이유]]



**출처 : 유튜브 쉬운코드 https://www.youtube.com/@ezcd
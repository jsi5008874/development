  

-사용자가 정의한 함수

-DBMS에 저장되고 사용되는 함수

-SQL의 select, insert, update, delete에서 사용할 수 있다.

  

### 예제1

조건 : 임직원의 ID를 열자리 정수로 랜덤하게 발급, ID의 맨 앞자리는 1로 고정

  

![[stored_function.png]]

delimiter $$ >> 이거는 함수의 마무리 표시인 ; 를 잠시 $$로 바꿔준거임

바꿔 준 이유는 create function에서 함수를 저장하는데 여기서도 ;가 필요함

그런데 delimiter를 안바꿔주고 ;를 그대로 사용하면 저장하는 함수에서 사용한 ;을 Create문이 끝나는걸로 인식해서 오류가 생김  
따라서 delimiter를 $로 바꿔 준 후 create function이 끝나면 다시 ;으로 바꿔주는거임  

  

1. Create Function 명령어 후 함수의 이름을 정해줌(id_generator())

1. 해당 함수의 Return 타입을 정해줌(int)
2. No SQL은 패스…
3. BEGIN으로 함수의 정의를 시작
4. RETURN 이후에 함수를 정의
5. END로 함수 정의 종료

  

![[stored_function2.png]]

이후에 활용은 이런식으로 필요한 부분에 데이터 값 대신 함수명을 넣어주면 된다.

  

  

### 예제2

조건 : 부서의 ID를 파라미터로 받으면 해당 부서의 평균 연봉을 알려주는 함수를 저장

  

![[stored_function3.png]]

함수명에 보면 파라미터가 들어가 있음 >> d_id == 파라미터 명, int == 파라미터의 타입

  

1. DECLARE는 변수를 선언하는 것이고 avg_sal int는 변수명과 타입이다.
2. select문에 into avg_sal은 select문으로 조회한 결과를 avg_sal 변수에 주입시키는 것
3. where 절에 dept_id = d_id 부분은 함수의 파라미터를 활용하여 조회할 때 사용하는 조건

  

![[stored_function4.png]]

DECLARE를 선언하지 않고 변수명 옆에 @를 사용하여 표현할 수 있다.

  

### 예제3

조건 : 졸업 요건 중 하나인 토익 800 이상을 충족했는지 알려주는 함수

  

![[stored_function5.png]]

IF 함수를 활용

  

### Stored Function 삭제하기

DROP FUNCTION 함수명;

  

### 등록된 Stored Function 파악하기

  

![[stored_function6.png]]

SHOW FUNCTION SATUS 구문을 활용해서 원하는 DB 내의 함수를 조회

  

![[stored_function7.png]]

함수명 조회 후 SHIOW CREATE FUNCTION 구문을 활용해 해당 함수의 코드를 확인

여기서 DEFINER는 해당 함수를 작성한 작성자의 정보임

  

### Stored Function은 언제 써야하는가?

Util 함수로 쓰기에는 좋지만 비즈니스 로직이 포함된 것은 안쓰는것이 좋다

ex) 예제3과 같이 비즈니스 로직이 들어가면 Stored Function으로 관리하는 것 보다는

Spring에서 비즈니스 로직으로 구현하는 것이 좋고

예제2와 같이 간단한 계산 등 Util 함수들은 Stored Function으로 처리해도 좋다.

  

![[stored_function8.png]]

**출처 : 유튜브 쉬운코드 https://www.youtube.com/@ezcd
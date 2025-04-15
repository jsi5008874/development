  

데이터 변경이 생겼을 때 즉, DB에 insert, update, delete가 발생했을 때

이것이 계기가 되어 자동적으로 실행되는 프로시저를 의미

  

### 예제1

사용자의 닉네임 변경 이력을 저장하는 트리거 작성

  

![[image 65.png]]

코드마스터라는 닉네임을 쉬운코드로 바꾸는 업데이트가 발생하면

기존의 코드마스터 닉네임을 Users_Log 테이블에 저장시켜주는 트리거

  

![[image 66.png]]

1. CREATE TRIGGER 트리거 명 작성
2. BEFORE UPDATE >> 업데이트가 발생하기 전에 트리거 작동
3. ON 테이블명 FOR EACH ROW >> 해당 테이블에 업데이트된 로우에 적용
4. 프로시저 작성
5. OLD는 Update 되기 전 원래의 데이터를 의미(delete였다면 delete 전 데이터)

  

### 예제2

사용자가 마트에서 상품을 구매할 때마다 지금까지 누적된 구매비용을 구하는 트리거 작성

  

![[image 67.png]]

동일한 아이디의 user가 구매를 하면 user_buy_stats 테이블에 누적

  

![[image 68.png]]

1. 트리거 생성
2. After Insert >> insert 발생 후에 프로시저 실행
3. Decalre user_id int Default NEW.user_id >> user_id 변수를 선언하는데 insert된 후의 데이터 값으로 초기화 해주는 것임
4. 프로시저 내용 작성

  

### trigger를 정의할 때 알면 좋은 내용

1. update, insert, delete 등을 한번에 감지하도록 설정가능

(mysql은 불가능)

![[image 69.png]]

이렇게 or를 써서 한번에 감지하도록 만들 수 있다.

FOR EACH ROW를 하면 1003 부서에 대한 트리거도 다섯번 실행되서 비효율적

  

![[image 70.png]]

FOR EACH STATEMENT를 사용하면 한번만 실행가능

(mysql은 불가능)

  

1. trigger를 발생시킬 디테일한 조건을 지정할 수 있다.

(mysql 불가능)

![[image 71.png]]

이렇게 WHEN을 써서 특별한 조건을 걸어서 만들 수 있다.

  

### Trigger 사용 시 주의사항

1. 소스 코드로는 발견할 수 없는 로직이기 떄문에 어떤 동작이 일어나는지 파악하기 어렵고 문제가 생겼을 때 대응하기 어렵다

  

1. 트리거를 너무 많이 생성하면 트리거 연쇄작용이 일어나서 원치 않는 데이터까지 변경될 수 있다.

  

1. 과도한 트리거 사용은 DB에 부담을 주고 응답을 느리게 만든다.

  

1. 디버깅이 어렵다.

  

1. 문서 정리가 특히나 중요하다.

  

결론 : 트리거는 최후의 수단으로…



**출처 : 유튜브 쉬운코드 https://www.youtube.com/@ezcd
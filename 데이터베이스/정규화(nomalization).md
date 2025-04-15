  

데이터 중복과 insertion, update, deletion anomaly를 최소화하기 위해

일련의 normal form(NF)에 따라 relational DB를 구성하는 과정

  

normal form(NF) : 정규화 되기 위해 준수해야 하는 몇 가지 rule

  

  

![[image 153.png]]

정규화에는 각 단계가 있으며 단계를 만족해야 다음 단계로 진행 가능

  

![[image 154.png]]

1~BCNF까지가 FD와 key만으로 정의되는 normal form이다.

  

보통 3NF까지 많이 하고 많이 해야 4NF까지만 진행한다.

5, 6NF는 해당 케이스가 발생할 확률도 적고 정규화도 굉장히 복잡하다.

5, 6NF는 거의 학술적 측면의 개념이라 생각하면 된다.

  

  

### 예제 스키마 정의

  

![[image 155.png]]

임직원 월급 계좌를 관리하는 테이블을 정의

  

![[image 156.png]]

해당 테이블의 key를 정의해보면

super key : tuple(row)를 유니크하게 식별할 수 있는 attribute set

(candidate key) : 한 attribute가 제거되면 유니크하게 식별할 수 없는 super key

  

![[image 157.png]]

primary key : 테이블에서 튜플들을 유니크하게 식별하려고 선택된 key를 의미

여기서는 account_id가 된다.

prime attribute : 임의의 key에 속하는 attribute

non-prime attribute : 어떠한 key에도 속하지 않는 attribute

  

  

(candidate) key는 위의 예시와 같이 account_id가 될 수 있고

bank_name, account_num이 될 수 있다.

account_num만 지정할 수 없는 이유는 은행이 달라도 계좌번호가 같을 수 도 있기 때문에

둘 중 하나가 제거되면 유니크하지 않아서 key로 사용

  

  

### FD(Functional Dependency) 관점에서의 분석

![[image 158.png]]

프라이머리 키 기준으로

left hand side에 primary key, right hand side에 다른 attribute가 있다.

즉 primary key를 알면 다른 attribute의 값도 알 수 있다는 뜻

이를 도식화하면 위 처럼 PK에는 실선, 나머지에는 화살표가 들어간다

  

![[image 159.png]]

여기서 아까 정의할 때 각 은행별 계좌의 등급이 있었는데

계좌 등급의 이름들이 겹치지 않는 것을 볼 수 있다.

그렇다면 class의 이름을 가지고 bank_name을 알 수 있다는 뜻이다.

  

![[image 160.png]]

이렇게 class → bank_name을 추가 할 수 있다.

  

  

### 1NF 정규화

![[image 161.png]]

1NF는 반드시 나눠질 수 없는 단일 값이어야한다는 규칙인데

messi의 card_id를 보면 두 개가 있다.

  

![[image 162.png]]

이렇게 수정을 하면 1NF를 만족하지만 문제가 있다.

PK인 account_id에 a21이라는 중복 값이 생겼고

card_id를 제외한 모든 값이 중복되는 튜플도 하나가 더 추가되었다.

또한 ratio도 합해서 1이 되어야하는데 그대로 복사해서 분리하다보니 ratio의 일관성에도 문제 발생

  

따라서 PK를 변경해야하는데 account_id와 card_id 두개를 중복 PK로 설정한다.

  

  

### 2NF 정규화

모든 non-prime attribute는 모든 key에 fully functionally dependent 해야한다.

>> 부분 종속이 아닌 완전 종속이 되어야 한다는 뜻

  

중복데이터가 생긴 이유는 무엇일까?

  

![[image 163.png]]

우선 PK를 변경하면서 key가 위와 같이 바뀌었다.

account_id만으로 유니크하게 식별 불가해서 card_id가 추가 되었고

messi의 card_id를 쪼개면서 중복 튜플이 생겼는데 이 때문에 bank_name, account_num만으로는

유니크하게 식별이 불가능해서 card_id를 추가해줬다.

  

non-prime attribute는 key를 제외한 모든 attribute이다.

  

![[image 164.png]]

여기서 현재 테이블의 특징을 보면

모든 non-prime attribute들이 key에 partially dependent하다.

>> 사실 account_id만 있어도 non-prime attribute들을 식별할 수 있다는 뜻이다.

여기서 card_id 없이 account_id 만으로도 class, ratio, empl_id, empl_name을 식별할 수 있어서

fully가 아닌 partially 즉, 완전 종속이 아닌 부분 종속인 것이다.

  

![[image 165.png]]

또한 다른 key인 bank_name, account_num, card_id에서도

bank_name, account_num만 있어도 모든 non-prime attribute를 식별 할 수 있어서

부분 종속이 된다.

  

그래서 현재 테이블 구조는 2NF의 규칙인

‘모든 non-prime attribute는 모든 key에 fuuly functionally dependent 해야한다.‘를 위배한 상태이다.

  

![[image 166.png]]

최종적으로는 card_id를 따로 빼서 테이블을 새로 만들고

나중에 join을 위해 account_id를 PK로 작성

  

또한 card_id 때문에 중복 값이 있던 messi의 튜플도 card_id를 빼면 완전히 같은 값이기 때문에

삭제해주고 한 개만 남겨둔다.

  

이렇게 되면 card_id 때문에 부분 종속이 되었던 것이 모두 완전 종속으로 바뀌면서 2NF를 만족하게된다.

  

### 3NF 정규화

![[image 167.png]]

먼저 FD를 파악

1. empl_id → empl_name >> empl_id의 값에 따라 empl_name이 결정된다.
2. account_id → empl_id >> account_id의 값에 따라 empl_id가 결정된다.

FD는 순간의 상태를 보는게 아니라 attribute의 관계를 보고 파악하는 것

즉, a11 > e1, a12 > e1, a13 >e1 셋 다 e1인데 그럼 account_id에 따라 empl_id가 정해지는게 아니지 않나? 라고 생각하는게 아니고

account_id는 계좌 id이고 계좌 id를 통해 empl_id 즉 계좌의 소유자인 임직원이 누군지 식별할 수 있으므로 FD가 성립되는 것이다.

  

이처럼 account_id → empl_id → empl_name이 되므로 account_id → empl_name이 된다.

  

![[image 168.png]]

이와 같은 상황(의존에 의존이 겹친 상황)을 transitive FD라고 한다.

  

![[image 169.png]]

하지만 Y 또는 Z가 어떠한 키의 부분 집합이 아니어야 한다는 조건이 있다.

  

ex) account_id → class, class → bank_name이므로 account_id → bank_name이 성립되어야 하지만

bank_name은 {bank_name, account_num} 키의 부분 집합이어서 transitive FD가 되지 못함

  

  

3NF의 Rule

![[image 170.png]]

non-prime attribute와 non-prime attribute 사이에는 FD가 있으면 안된다.

하지만 지금 empl_id, empl_name 사이에 FD가 존재하므로 3NF 규칙 위반

  

![[image 171.png]]

3NF까지의 최종 모습

  

  

### BCNF 정규화

  

![[image 172.png]]

  

non-trivial Fd는 X→Y일 때 Y가 X의 부분 집합이 아닌 경우를 의미한다.

또한 X가 슈퍼키여야하는데 현재 테이블에서 class는 슈퍼키가 아니므로 BCNF 규칙을 위반

  

![[image 173.png]]

BCNF 적용 후 모습



**출처 : 유튜브 쉬운코드 https://www.youtube.com/@ezcd
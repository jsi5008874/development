  

  

### 정의 : 한 테이블에 있는 두 개의 attribute(s) 집합(set) 사이의 제약(constraint)

  

  

![[image 136.png]]

X의 값에 따라 Y의 값이 변하는 것임

예를 들어 김주성이라는 사원의 id가 1이고 이찬희라는 사원의 id가 2일 때

1로 검색하면 1에 따라 김주성에 관한 정보가 나오고

2로 검색하면 2에 따라 이찬희의 정보가 나온다

  

![[image 137.png]]

이런 관계를 functional dependency(FD) 라고 부르고

기호로는 X → Y 로 표시한다.

  

### FD 파악하기

테이블의 스키마를 보고 의미적으로 파악해야 한다.

즉, 테이블의 state를 보고 FD를 파악해서는 안된다.

  

![[image 138.png]]

테이블의 순간 특정 상태만 보고 결정하면 안된다는 소리

예를 들어 empl_name → birth_date로 가정했다면 위와 같이 동명이인이 있을 때는?

정확한 정보를 얻을 수 없게된다.

이게 state를 보고 FD를 파악하면 안된다는 뜻이다.

  

![[image 139.png]]

즉 테이블의 스키마를 보고 의미적으로 FD를 파악

현재 예시에서 empl_name → birth_date로 생각해본다면 동명이인이 있을 수 있기 때문에

FD로 판단하면 안된다고 판단해야한다.

  

**즉 스키마의 특성을 통해 의미를 생각해서 판단**

  

  

![[image 140.png]]

만약 집합 Y가 dept_id까지 포함하는데 임직원이 하나 이상의 부서에 속할 수 있다면

x→ y가 유효한가?

답은 아니다.

만약 김주성이라는 사원이 개발팀, 운영팀에 소속되어 있다면

empl id =1로 검색을 하더라도 개발팀 튜블, 운영팀 튜블이 조회될 것이다.

따라서 구축하려는 DB의 attribute가 관계적으로 어떤 의미를 지닐지에 따라 FD가 달라진다.

  

![[image 141.png]]

left, right로 나눠서 표현을 한다.

  

FD의 예시

![[image 142.png]]

  

  

### FD에서 공집합

![[image 143.png]]

공집합 → 집합의 형태라면 right-hand side의 집합은 언제나 하나의 값을 가진다는 의미이다.

  

![[image 144.png]]

이렇게 attribute의 값이 모두 동일하다면 해당되는 표현이다.

  

  

### Trivial Functional Dependency

![[image 145.png]]

Y가 X의 부분 집합일 경우 trivial FD라고 한다.

  

### **NON-Trivial Functional Dependency**

![[image 146.png]]

completely non-trivial FD는 전혀 겹치는 부분이 하나도 없을 때

  

### Partial Functional Dependency

  

![[image 147.png]]

X의 proper subset이 Y를 결정할 때 그건 partial FD

  

proper subset이란

![[image 148.png]]

즉 집합 X와 완전이 일치하지 않는 부분 집합을 의미

  

![[image 149.png]]

이런 경우 empl_id만 있어도 birth_date를 결정할 수 있으므로 patial FD가 된다.

  

### Full functional dependency

![[image 150.png]]

X의 proper subset이 Y를 결정할 수 없을 때 full FD가 된다.

예시를 통해 보면 grade는 어떤 학생이 어떤 수업에서 받은 성적을 의미하는데

예를 들어 김주성의 과학 성적을 알고싶으면 김주성의 stu_id, 과학의 class_id를 통해

김주성의 과학 성적을 조회하는것인데

김주성의 stu_id 만으로는 과학 성적을 알 수 없고, class_id만으로는 김주성의 성적을 알 수 없음

  

이런 경우를 Full FD라고 한다.



**출처 : 유튜브 쉬운코드 https://www.youtube.com/@ezcd
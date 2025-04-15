  

JpaRepository의 save()는 DB에 새로 삽입하거나 수정할 때 사용

그런데 EntityManager도 persist()로 삽입하고 merge()로 수정을 하는데

둘의 차이는 뭘까?

  ![[Pasted image 20250404175029.png]]

### JpaRepository
SpringataJPA가 제공하는 자동 구현 인터페이스
save(), findById() 등 기본 CRUD 자동 구현
**빠르게 개발할 수 있지만 복잡한 쿼리는 부적합하다.**
  
### EntityManager
JPA의 핵심 객체로 직접 DB와 상호작용할 수 있는 API
persist(), find() 등 객체 직접 조작
JPQL, Native Query 등 jpa 문법에 맞는 쿼리 작성 가능


**JpaRepository는 Spring에서 지원하는 기능이고 EntityManager는 Java의 고유 API이다.**

jpaRepository는 내부적으로 EntityManager를 사용하고 있다.


![[do-messenger_screenshot_2024-11-27_16_46_30.png]]

  

JpaRepository의 save()를 보면 결국 entityManager의 persist()와 merge() 메서드를 사용 중이다.

즉 JpaRepostory가 entityManger의 메서드를 호출해서 사용 중인 것이다.
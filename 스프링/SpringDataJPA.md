  

**JPA 위에서 동작하는 Spring Framework 모듈로, JPA를 더 쉽게 사용할 수 있도록**

**추상화 및 자동화를 제공**

**JPA 구현체(Hibernate 등)를 내부적으로 사용**

  

### **특징**

1. **JPA를 더 쉽게 사용**:
    - Repository 계층의 반복적인 코드를 줄이고, 데이터 접근 로직을 자동 생성.
    - 예: 쿼리 메서드, 메서드 이름으로 자동으로 JPQL 생성.
2. **Spring과의 통합**:
    - Spring DI, 트랜잭션 관리(@Transactional), AOP와 자연스럽게 연동.
3. **추상화된 Repository**:
    - `**JpaRepository**`, `**CrudRepository**` 등 기본 인터페이스 제공.
    - 커스텀 쿼리 작성도 가능 (JPQL, Native Query, QueryDSL 지원).
4. **프로덕션 생산성 향상**:
    - 간단한 메서드 정의만으로 CRUD, 페이징, 정렬 기능 제공.
5. **동작 원리**:
    - 내부적으로 JPA 구현체를 사용하며, Spring의 의존성 주입으로 개발자 친화적인 환경 제공.

  

List<User> findByName(String name);

**이런 쿼리메서드로 쿼리를 자동 생성**

하지만 복잡한 쿼리의 경우에는 쿼리 메서드는 좋지 않아서

JPQL, Native Query, QueryDSL 같은 기능을 사용한다.
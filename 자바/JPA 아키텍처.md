  

![[image 309.png]]

  

1. **EntityManagerFactory**

- EntityManager 클래스의 팩토리 클래스입니다. 이 클래스로 EntityManager 클래스의 인스턴스를 생성하고 관리할 수 있습니다.

1. **EntityManager**

- 인터페이스입니다. 객체에 대한 영속성 관리작업을 합니다. Query 인스턴스를 생성하는 팩토리처럼 작동합니다.

1. **Entity**

- 영속객체입니다. 이 객체의 스펙에 의해서 데이터베이스에 기록될 객체입니다.

1. **EntityTransaction**

- EntityManager와 일대일 관계입니다. 각각의 EntityManager들의 작업은 EntityTransaction 클래스에 의해서 유지됩니다.

1. **Persistence**

- 이 클래스는 EntityManagerFactory 인스턴스를 생성하는 정적(static) 메소드를 가지고 있습니다.

1. **Query**

- 인터페이스로서 각각의 JPA 벤더에 의해 구현되며 각 기준에 충족하는 관계형 객체를 얻습니다.

  

**JPA 아키텍처의 작동 흐름**

1. Persistence가 EntityManagerFactory 생성
2. EntityManagerFactory가 EntityManager 생성
3. EntityManager가 Entity를 관리
4. EntityManager는 EntityTransaction과 일대일 매핑이 되서 EntityManager가 하는 작업을

EntityTransaction에서 관리

  

현재는 여기까지 생각할 수 있고 더 자세한 동작방식을 알기위해서는 EntityManager가

무엇을 하는지 알아야한다.

  

**EntityManager는 Entity의 생명주기를 관리하고 영속성 컨텍스트에 저장해서 관리를 한다.**

- 주요 기능:
    - 엔티티의 **CRUD** 작업 수행
    - 영속성 컨텍스트와의 상호작용
    - 트랜잭션 처리

  

**Entity의 생명주기**

![[image 310.png]]

  

1. Transient(비영속 상태)

Entity가 생성되고 Entity Manager에게 등록되지 않은 상태

1. Managed(영속 상태)

Entity가 영속성 콘텍스트에 의해 관리되는 상태

1. Detached(준영속 상태)

Entity가 영속성 콘텍스트에 저장되었다가 분리된 상태

1. Removed(삭제 상태)

DB에 저장되어있는 Entity가 삭제된 상태

  

### **영속성 컨텍스트**

JPA에서 엔티티를 **1차 캐시**로 관리하는 메모리 공간  
엔티티 매니저가 엔티티를 관리하기 위해 사용하는 작업 영역  

  

**역할 :**

1. **1차 캐시 역할**:

- 엔티티를 메모리 상에 저장하고 관리합니다.
- 동일한 엔티티를 여러 번 조회하면 데이터베이스가 아닌 1차 캐시에서 가져옵니다.
- 동일한 트랜잭션 내에서 동일한 엔티티 객체에 대해 동일성을 보장합니다.

1. **변경 감지 (Dirty Checking)**:
    - 엔티티의 상태 변화를 감지하여 트랜잭션 종료 시점에 변경 내용을 데이터베이스에 반영합니다.
2. **CRUD 작업의 중재자 역할**:
    - `**EntityManager**`가 수행하는 명령(persist, find, merge, remove 등)에 따라 영속성 컨텍스트가 엔티티를 관리합니다.

  

영속성 컨텍스트는 실제 엔티티 객체를 관리하는 저장소 역할을 하고

**EntityManager**는 이를 제어하는 도구

  

**실제로 DB와 연결되서 쿼리를 실행하는 것은 JPA 구현체인 Hibernate가 담당**

**>> EntityManager가 영속성 컨텍스트에게 명령(persist, find, merge, remove등)을 내리면**

**영속성 컨텍스트는 그 명령에 따라 Entity의 상태를 저장하고**

**commit 명령이 내려지면 Hibernate에게 쿼리 실행을 요청한다.**

  

[[SpringDataJPA]]
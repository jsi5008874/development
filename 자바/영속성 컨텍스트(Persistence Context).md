  

JPA에서 **엔티티 객체를 영속 상태로 관리하는 일종의 캐시 영역**

JPA가 **데이터베이스와의 작업을 효율적으로 관리하기 위해 엔티티 객체를 메모리에 보관하는 곳**

  

1. **엔티티 관리**: 영속성 컨텍스트는 엔티티 객체를 저장하고 관리합니다. 특정 엔티티가 영속성 컨텍스트에 등록되면 '영속 상태'로 전환되며, 이 상태에서는 JPA가 엔티티의 상태 변화를 자동으로 추적합니다.
2. **1차 캐시**: 영속성 컨텍스트는 **1차 캐시** 역할을 수행합니다. 같은 엔티티를 반복 조회할 경우, 데이터베이스에 바로 접근하는 대신 영속성 컨텍스트에 있는 데이터를 반환합니다.
3. **변경 감지**: 트랜잭션이 끝나기 전까지 영속성 컨텍스트는 엔티티의 변경을 감지합니다. 변경된 엔티티는 트랜잭션이 끝나기 전에 자동으로 데이터베이스에 반영됩니다.
4. **쓰기 지연**: 여러 엔티티에 대해 수정 작업이 발생할 때, **바로 데이터베이스에 반영하지 않고** 모아서 저장해 효율적으로 관리할 수 있습니다.
5. **동일성 보장**: 영속성 컨텍스트 내에서는 동일한 엔티티를 조회할 경우, **동일한 객체**를 반환하여 객체의 일관성을 보장합니다.

  

### 예시

```Java
@Entity
public class Member {
@Id
private Long id;
private String name;
}

@Transactional
public void updateMember(Long memberId, String newName) {
// 1. find() 메서드를 호출하여 영속성 컨텍스트에 저장
Member member = entityManager.find(Member.class, memberId);
// 2. 이 시점에 member는 영속 상태가 됨
member.setName(newName); // 변경 감지로 인해 영속성 컨텍스트가 변화 추적

// 3. 트랜잭션 종료 시점에 변경 내용을 데이터베이스에 반영 (UPDATE 쿼리 실행)
}
```

  

여기서 `**member**` 객체는 `**find**` 메서드를 통해 영속성 컨텍스트에 저장되며, 이후 `**setName()**`으로 값을 변경하면 JPA가 이를 감지하고 트랜잭션 종료 시 자동으로 업데이트  
  

### 엔티티는 언제 영속성 컨텍스트에서 제거 되는가?

  

트랜잭션이 종료되면 엔티티 객체는 영속성 컨텍스트에서 제거

이를 **'준영속(detached) 상태'** 라고 부르며, 영속성 컨텍스트가 더 이상 객체의 상태 변화를 추적하지 않는 상태  
  

  
영속성 컨텍스트에서의 상태 변화  

1. **영속 상태 (Persistent)**: 트랜잭션 내에서 영속성 컨텍스트에 관리되는 상태로, 엔티티 객체는 JPA에 의해 변경 사항이 추적됩니다.
2. **준영속 상태 (Detached)**: 트랜잭션이 끝나거나 영속성 컨텍스트가 닫히면, 영속 상태였던 엔티티는 더 이상 관리되지 않고 준영속 상태가 됩니다.
3. **삭제 상태 (Removed)**: 영속성 컨텍스트에서 완전히 제거된 상태로, `**remove()**` 메서드를 사용해 엔티티를 삭제할 수 있습니다. 트랜잭션이 끝나면 `**DELETE**` 쿼리가 실행됩니다.

  

예시

```Java
@Transactional
public void someMethod() {
Member member = entityManager.find(Member.class, memberId); // 영속 상태
member.setName("New Name"); // 변경 감지
// 트랜잭션 종료 (영속성 컨텍스트가 닫힘)
}
// 트랜잭션 밖에서 변경
member.setName("Another Name"); // 데이터베이스에 반영되지 않음
```

  

다시 영속성 컨텍스트로 포함하는 방법 : merge()

  

```Java
@Transactional
public void updateMember(Member member) {
    Member managedMember = entityManager.merge(member);
    managedMember.setName("Updated Name");

*// 변경 감지로 업데이트 반영*

}
```

이렇게 하면 `**merge()**`로 반환된 `**managedMember**` 객체가 영속 상태가 되어, 변경 사항이 데이터베이스에 반영됩니다.  
  

###   
@Transactional을 사용하지 않았을 경우 영속성 컨텍스트는 언제 제거 되는가?  
  

영속성 컨텍스트는 Entity Manager를 통해 관리가 되는데 @Transactional 어노테이션이 없는

경우에는 Entity Manager가 명시적으로 종료될 때 제거 된다.

  

  
• **EntityManager.close() 호출**: 영속성 컨텍스트는 `**EntityManager.close()**`가 호출될 때 소멸됩니다. `**EntityManager**`가 닫히면, 이 객체에 대한 추적을 중지하고, 더 이상 데이터베이스와 연결된 상태가 아닙니다.

  

  

  

[[Entity Manager]]
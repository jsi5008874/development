  

**null or 값을 감싸서 NPE(NullPointerException)로부터 부담을 줄이기 위해 등장한 Wrapper 클래스**

  

### **Optional.empty() - 값이 Null인 경우**

Optional은 Wrapper 클래스이기 때문에 값이 없을 수도 있는데, 이때는 Optional.empty()로 생성할 수 있다.

```Java
Optional<String> optional = Optional.empty();

System.out.println(optional);// Optional.empty
System.out.println(optional.isPresent());// false
```

  

### **Optional.of() - 값이 Null이 아닌 경우**

만약 어떤 데이터가 절대 null이 아니라면 Optional.of()로 생성할 수 있다. 만약 Optional.of()로 Null을 저장하려고 하면 NullPointerException이 발생한다.

```Java
// Optional의 value는 절대 null이 아니다.
Optional<String> optional = Optional.of("MyName");
```

절대 null이 아닌 경우에는 굳이 optional을 쓸 필요가 없다.

왜냐하면 optional 자체가 null을 wrapper 클래스로 감싸고 null이면 다시 풀어서 대체하는 함수를 호출하는 방식인데 감싸고 푸는 자체가 오버헤드가 되어서 성능이 저하되기 때문이다.

  

### **Optional.ofNullable() - 값이 Null일수도, 아닐수도 있는 경우**

만약 어떤 데이터가 null이 올 수도 있고 아닐 수도 있는 경우에는 Optional.ofNullable로 생성할 수 있다. 그리고 이후에 orElse 또는 orElseGet 메소드를 이용해서 값이 없는 경우라도 안전하게 값을 가져올 수 있다.

```Java
// Optional의 value는 값이 있을 수도 있고 null 일 수도 있다.
Optional<String> optional = Optional.ofNullable(getName());
String name = optional.orElse("anonymous");// 값이 없다면 "anonymous" 를 리턴
```

  

  

### **orElse와 orElseGet의 차이**

Optional API의 단말 연산에는 orElse와 orElseGet 함수가 있다. 비슷해 보이는 두 함수는 엄청난 차이가 있다.

- orElse: 파라미터로 값을 받는다.
- orElseGet: 파라미터로 함수형 인터페이스(함수)를 받는다.

  

![[image 308.png]]

  

```Java
public final class Optional<T> {

    ...// 생략public T orElse(T other) {
        return value != null ? value : other;
    }

    public T orElseGet(Supplier<? extends T> other) {
        return value != null ? value : other.get();
    }

}
```

orElse로는 값이, orElseGet로는 함수가 넘어간다는 것은 상당히 큰 차이가 있다.

  

예시)

첫 번째 함수는 값이 비어있을 때 orElse를 호출하도록 되어있고, 두 번째 함수는 orElseGet을 호출하도록 되어있다. 바로 아래로 내려가지 않고, 각각에 의한 출력 결과를 예상해보도록 하자.

```Java
public void findUserEmailOrElse() {
    String userEmail = "Empty";
    String result = Optional.ofNullable(userEmail)
    	.orElse(getUserEmail());

    System.out.println(result);
}

public void findUserEmailOrElseGet() {
    String userEmail = "Empty";
    String result = Optional.ofNullable(userEmail)
    	.orElseGet(this::getUserEmail);

    System.out.println(result);
}

private String getUserEmail() {
    System.out.println("getUserEmail() Called");
    return "mangkyu@tistory.com";
}
```

결과값

```Java
// 1. orElse인 경우
getUserEmail() Called
Empty

// 2. orElseGet인 경우
Empty
```

  

OrElse인 경우에는 다음과 같은 순서로 처리가 된다.

1. Optional.ofNullable로 "EMPTY"를 갖는 Optional 객체 생성
2. getUserEmail()가 실행되어 반환값을 orElse 파라미터로 전달
3. orElse가 호출됨, "EMPTY"가 Null이 아니므로 "EMPTY"를 그대로 가짐

  

Optional.orElse()가 값을 파라미터로 받고, **orElse 파라미터로 값을 넘겨주기 위해 getUserEmail()이 호출되었기 때문**

  

하지만 함수형 인터페이스(함수)를 파라미터로 받는 orElseGet에서는 동작이 달라진다.

1. Optional.ofNullable로 "EMPTY"를 갖는 Optional 객체 생성
2. getUserEmail() 함수 자체를 orElseGet 파라미터로 전달
3. orElseGet이 호출됨, "EMPTY"가 Null이 아니므로 "EMPTY"를 그대로 가지며 getUserEmail()이 호출되지 않음

orElseGet에서는 파라미터로 넘어간 값인 getUserEmail 함수가 Null이 아니므로 .get에 의해 함수가 호출되지 않는다. 만약 Optional의 값으로 null이 있다면, 다음과 같은 흐름에 의해 orElseGet의 파라미터로 넘어온 getUserEmail()이 실행될 것이다.

```Java
public void findUserEmailOrElseGet() {
    String result = Optional.ofNullable(null)
    	.orElseGet(this::getUserEmail);

    System.out.println(result);
}

private String getUserEmail() {
    System.out.println("getUserEmail() Called");
    return "mangkyu@tistory.com";
}
```

1. Optional.ofNullable로 null를 갖는 Optional 객체 생성
2. getUserEmail() 자체를 orElseGet 파라미터로 전달
3. orElseGet이 호출됨, 값이 Null이므로 other.get()이 호출되어 getUserEmail()가 호출됨

  

### **orElse에 의한 발생가능한 장애 예시**

위에서 살펴보았듯 orElse와 orElseGet은 명확하고 중요한 차이가 있는데, 이를 정확히 인식하지 못하면 장애가 발생할 수 있다. 예를 들어 userEmail을 Unique한 값으로 갖는 시스템에서 아래와 같은 코드를 작성하였다고 하자.

```Java
public void findByUserEmail(String userEmail) {
// orElse에 의해 userEmail이 이미 존재해도 유저 생성 함수가 호출되어 에러 발생return userRepository.findByUserEmail(userEmail)
            .orElse(createUserWithEmail(userEmail));
}

private String createUserWithEmail(String userEmail) {
    User newUser = new User(userEmail);
    return userRepository.save(newUser);
}
```

위의 예제는 Optional의 **단말 연산**으로 orElse를 사용하고 있기 때문에, 조회 결과와 무관하게 createUserWithEmail 함수가 반드시 실행된다. 하지만 Database에서는 userEmail이 Unique로 설정되어 있기 때문에 오류가 발생할 것이다. 그렇기 때문에 위와 같은 경우에는 다음과 같이 해당 코드를 orElseGet으로 수정해야 한다. 이렇게 코드를 수정하였다면 파라미터로 createUserWithEmail 함수 자체가 넘어가므로, 조회 결과가 없을 경우에만 사용자를 생성하는 로직이 호출 될 것이다.

  

### **[ Optional이 위험한 이유, Optional을 올바르게 사용해야 하는 이유 ]**

Optional을 사용하면 코드가 Null-Safe해지고, 가독성이 좋아지며 애플리케이션이 안정적이 된다는 등과 같은 얘기들을 많이 접할 수 있다. 하지만 이는 Optional을 목적에 맞게 올바르게 사용했을 때의 이야기이고, Optional을 남발하는 코드는 오히려 다음과 같은 부작용(Side-Effect)를 유발할 수 있다.

- NullPointerException 대신 NoSuchElementException가 발생함
- 이전에는 없었던 새로운 문제들이 발생함
- 코드의 가독성을 떨어뜨림
- 시간적, 공간적 비용(또는 오버헤드)이 증가함

  

### **NullPointerException 대신 NoSuchElementException가 발생함**

만약 Optional로 받은 변수를 값이 있는지 판단하지 않고 접근하려고 한다면 NoSuchElementException가 발생하게 된다.

```SQL
Optional<User> optionalUser = ... ;

// optional이 갖는 value가 없으면 NoSuchElementException 발생
User user = optionalUser.get();
```

Null-Safe하기 위해 Optional을 사용하였는데, 값의 존재 여부를 판단하지 않고 접근한다면 NullPointerException는 피해도 NoSuchElementException가 발생할 수 있다.

### **이전에는 없었던 새로운 문제들이 발생함**

Optional을 사용하면 문제가 되는 대표적인 경우가 직렬화(Serialize)를 할 때이다. 기본적으로 Optional은 직렬화를 지원하지 않아서 캐시나 메세지큐 등과 연동할 때 문제가 발생할 수 있다. Optional을 사용함으로써 오히려 새로운 문제가 발생할 수 있는 것이다.

```Java
public class User implements Serializable {

    private Optional<String> name;

}
```

물론 Jackson처럼 라이브러리에서 Optional이 있을 경우 값을 wrap, unwrap 하도록 지원해주기도 하지만 해당 라이브러리의 스펙을 파악해야한다는 점 등에서 오히려 불편함을 느낄 수 있다.

참고로 Jackson 라이브러리는 jackson-modules-java8 project 프로젝트부터 Optional을 지원하여, empty일 경우에는 null을 값이 있는 경우에는 값을 꺼내서 처리하도록 지원하고 있다. 또한 Cacheable, CacheEvict, CachePut과 같은 Spring의 캐시 추상화 기술들은 Spring4부터 이러한 wrap, unwrap을 지원하고 있다.

### **코드의 가독성을 떨어뜨림**

예를 들어 다음과 같이 Optional을 받아서 값을 꺼낸다고 하자. 우리는 앞에서 살펴본대로 optionalUser의 값이 비어있으면 NoSuchElementException가 발생할 수 있으므로 다음과 같이 값의 유무를 검사하고 꺼내도록 코드를 작성하였다.

```C++
public void temp(Optional<User> optionalUser) {
    User user = optionalUser.orElseThrow(IllegalStateException::new);

// 이후의 후처리 작업 진행...
}
```

그런데 위와 같은 코드는 또 다시 NPE를 유발할 수 있다. 왜냐하면 optionalUser 객체 자체가 null일 수 있기 때문이다. 그러면 우리는 이러한 문제를 해결하기 위해 다음과 같이 코드를 수정해야 한다.

```TypeScript
public void temp(Optional<User> optionalUser) {
    if (optionalUser != null && optionalUser.isPresent()) {
// 이후의 후처리 작업 진행...
    }

    throw new IllegalStateException();
}
```

이러한 코드는 오히려 값의 유무를 2번 검사하게 하여 단순히 null을 사용했을 때보다 코드가 복잡해졌다. 그 외에도 Optional의 제네릭 타입 때문에도 불필요하게 코드 글자수까지 늘어났다. 이렇듯 Optional을 올바르게 사용하지 못하고 남용하면 오히려 가독성이 떨어질 수 있다.

### **시간적, 공간적 비용(또는 오버헤드)이 증가함**

- 공간적 비용: Optional은 객체를 감싸는 컨테이너이므로 Optional 객체 자체를 저장하기 위한 메모리가 추가로 필요하다.
- 시간적 비용: Optional 안에 있는 객체를 얻기 위해서는 Optional 객체를 통해 접근해야 하므로 접근 비용이 증가한다.

어떤 글에서는 Optional을 사용하면 단순 값을 사용했을 때보다 메모리 사용량이 4배 정도 증가한다고 한다. 그 외에도 Optional은 Serializable 하지 않는 등의 문제가 있으므로 이를 해결하기 위해 추가적인 개발 시간이 소요

  

### **[ 올바른 Optional 사용법 가이드 ]**

- Optional 변수에 Null을 할당하지 말아라
- 값이 없을 때 Optional.orElseX()로 기본 값을 반환하라
- 단순히 값을 얻으려는 목적으로만 Optional을 사용하지 마라
- 생성자, 수정자, 메소드 파라미터 등으로 Optional을 넘기지 마라
- Collection의 경우 Optional이 아닌 빈 Collection을 사용하라
- 반환 타입으로만 사용하라

### **Optional 변수에 Null을 할당하지 말아라**

Optional은 컨테이너/박싱 클래스일 뿐이며, Optional 변수에 null을 할당하는 것은 Optional 변수 자체가 null인지 또 검사해야 하는 문제를 야기한다. 그러므로 값이 없는 경우라면 Optional.empty()로 초기화하도록 하자.

```C#
// AVOIDpublic Optional<Cart> fetchCart() {

    Optional<Cart> emptyCart = null;
    ...
}
```

### **값이 없을 때 Optional.orElseX()로 기본 값을 반환하라**

Optional의 장점 중 하나는 함수형 인터페이스를 통해 가독성좋고 유연한 코드를 작성할 수 있다는 것이다. 가급적이면 isPresent()로 검사하고 get()으로 값을 꺼내기 보다는 orElseGet 등을 활용해 처리하도록 하자.

```Java
private String findDefaultName() {
    return ...;
}

// AVOIDpublic String findUserName(long id) {

    Optional<String> optionalName = ... ;

    if (optionalName.isPresent()) {
        return optionalName.get();
    } else {
        return findDefaultName();
    }
}

// PREFERpublic String findUserName(long id) {

    Optional<String> optionalName = ... ;
    return optionalName.orElseGet(this::findDefaultName);
}
```

orElseGet은 값이 준비되어 있지 않은 경우, orElse는 값이 준비되어 있는 경우에 사용하면 된다. 만약 null을 반환해야 하는 경우라면 orElse(null)을 활용하도록 하자. 만약 값이 없어서 throw해야하는 경우라면 orElseThrow를 사용하면 되고 그 외에도 다양한 메소드들이 있으니 적당히 활용하면 된다.

추가적으로 Java9 부터는 ifPresentOrElse도 지원하고 있으며, Java 10부터는 orElseThrow()의 기본으로 NoSuchElementException()를 던질 수 있다. 만약 Java8이나 9를 사용중이라면 명시적으로 넘겨주면 된다.

### **단순히 값을 얻으려는 목적으로만 Optional을 사용하지 마라**

단순히 값을 얻으려고 Optional을 사용하는 것은 Optional을 남용하는 대표적인 경우이다. 이러한 경우에는 굳이 Optional을 사용해 비용을 낭비하는 것 보다는 직접 값을 다루는 것이 좋다.

```Java
// AVOIDpublic String findUserName(long id) {
    String name = ... ;

    return Optional.ofNullable(name).orElse("Default");
}

// PREFERpublic String findUserName(long id) {
    String name = ... ;

    return name == null
      ? "Default"
      : name;
}
```

### **생성자, 수정자, 메소드 파라미터 등으로 Optional을 넘기지 마라**

Optional을 파라미터로 넘기는 것은 상당히 의미없는 행동이다. 왜냐하면 넘겨온 파라미터를 위해 자체 null체크도 추가로 해주어야 하고, 코드도 복잡해지는 등 상당히 번거로워지기 때문이다. Optional은 반환 타입으로 대체 동작을 사용하기 위해 고안된 것임을 명심해야 하며, 앞서 설명한대로 Serializable을 구현하지 않으므로 필드 값으로 사용하지 않아야 한다.

```Java
// AVOIDpublic class User {

    private final String name;
    private final Optional<String> postcode;

    public Customer(String name, Optional<String> postcode) {
        this.name = Objects.requireNonNull(name, () -> "Cannot be null");
        this.postcode = postcode;
    }

    public Optional<String> getName() {
        return Optional.ofNullable(name);
    }

    public Optional<String> getPostcode() {
        return postcode;
    }
}
```

Optional을 접근자에 적용하는 경우도 마찬가지이다. 위의 예시에서 name을 얻기 위해 Optional.ofNullable()로 반환하고 있는데, Brian Goetz는 Getter에 Optional을 얹어 반환하는 것을 두고 남용하는 것이라고 얘기하였다.

> I think routinely using it as a return value for getters would definitely be over-use.- Brian Goetz(Java Architect) -

### **Collection의 경우 Optional이 아닌 빈 Collection을 사용하라**

Collection의 경우 굳이 Optional로 감쌀 필요가 없다. 오히려 빈 Collection을 사용하는 것이 깔끔하고, 처리가 가볍다.

```Java
// AVOIDpublic Optional<List<User>> getUserList() {
    List<User> userList = ...;// null이 올 수 있음return Optional.ofNullable(items);
}

// PREFERpublic List<User> getUserList() {
    List<User> userList = ...;// null이 올 수 있음return items == null
      ? Collections.emptyList()
      : userList;
}
```

아래의 경우도 사용을 피해야 하는 케이스이다. Optional은 비싸기 때문에 최대한 사용을 지양해야 한다. 아래의 케이스라면 map에 getOrDefault 메소드가 있으니 이걸 활용하는 것이 훨씬 좋다.

```Java
// AVOIDpublic Map<String, Optional<String>> getUserNameMap() {
    Map<String, Optional<String>> items = new HashMap<>();
    items.put("I1", Optional.ofNullable(...));
    items.put("I2", Optional.ofNullable(...));

    Optional<String> item = items.get("I1");

    if (item == null) {
        return "Default Name"
    } else {
        return item.orElse("Default Name");
    }
}

// PREFERpublic Map<String, String> getUserNameMap() {
    Map<String, String> items = new HashMap<>();
    items.put("I1", ...);
    items.put("I2", ...);

    return items.getOrDefault("I1", "Default Name");
}
```

### **반환 타입으로만 사용하라**

Optional은 반환 타입으로써 에러가 발생할 수 있는 경우에 결과 없음을 명확히 드러내기 위해 만들어졌으며, Stream API와 결합되어 유연한 체이닝 api를 만들기 위해 탄생한 것이다. 예를 들어 Stream API의 findFirst()나 findAny()로 값을 찾는 경우에 어떤 것을 반환하는게 합리적일지 Java 언어를 설계하는 사람이 되어 고민해보자. 언어를 만드는 사람의 입장에서는 Null을 반환하는 것보다 값의 유무를 나타내는 객체를 반환하는 것이 합리적일 것이다. Java 언어 설계자들은 이러한 고민 끝에 Optional을 만든 것이다.

그러므로 Optional이 설계된 목적에 맞게 반환 타입으로만 사용되어야 한다.
  

객체에서 어떤 Class를 참조해서 사용해야하는 상황이 생긴다면, 그 Class를 직접 참조하는 것이 아니라 그 **대상의 상위 요소(추상 클래스 or 인터페이스)로 참조**하라는 원칙  
  
  

클라이언트(사용자)가 상속 관계로 이루어진 모듈을 가져다 사용할때, 하위 모듈을 **직접 인스턴스를 가져다 쓰지 말라**는 뜻이다. 왜냐하면 그렇게 할 경우, 하위 모듈의 구체적인 내용에 클라이언트가 의존하게 되어 하위 모듈에 변화가 있을 때마다 **클라이언트나 상위 모듈의 코드를 자주 수정**해야 되기 때문이다.

따라서 한마디로 **상위의 인터페이스 타입의 객체로 통신하라는 뜻**

![[image 325.png]]

  

  

대표적으로 컬렉션 프레임워크를 들수 있는데, 보통 ArrayList 나 HashSet 자료형을 인스턴스화 할때 변수 타입을 ArrayList, HashSet 같은 구체 클래스 타입으로 선언하는 것이 아닌, List 나 Set 같은 인터페이스 타입으로 선언하는 것을 봐왔을 것이다.

이것도 DIP 원칙을 따른 코드 선언이라고 봐도 무방하다.

  

```Java
// 변수 타입을 고수준의 모듈인 인터페이스 타입으로 선언하여 저수준의 모듈을 할당
List<String> myList = new ArrayList()<>;
```

```Java
Set<String> mySet = new HashSet()<>;
```

```Java
Map<int, String> myMap = new HashMap()<>;
```

  

이처럼 자신보다 변하기 쉬운 것에 의존하던 것을 추상화된 인터페이스나 상위 클래스를 두어 변하기 쉬운 것의 변화에 영향받지 않게 하는 것이 의존 역전 원칙이다.  
상위 클래스일수록, 인터페이스일수록, 추상 클래스일수록 변하지 않을 가능성이 높기에 하위 클래스나 구체 클래스가 아닌 상위 클래스, 인터페이스, 추상 클래스를 통해 의존하라는 것이다.  

**그런데 결국 이 말은 추상화를 이용하라는 말과 일맥상통 한 것 같은데, 사실 의존 역전 역칙은 우리가 앞서 배운 개방-폐쇄 원칙과 긴밀한 관계가 있다.**

  

### **DIP 원칙 위반 예제**

RPG 게임에는 캐릭터가 장착할 수 있는 다양한 무기들이 존재한다. 다음과 같이 한손검, 양손검, 전투도끼, 망치 클래스가 있다고 가정

  

그리고 이러한 무기들을 장착할 Character 클래스가 있다.

이 캐릭터 클래스는 인스턴스화될때 캐릭터 이름과 체력 그리고 장착하고 있는 무기를 입력값으로 받아 초기화 한다.

한손검도 한가지만 있는게 아니라 강철검, 미스릴검 같이 다양한 타입의 검이 올수 있기 때문에 캐릭터 클래스내에 필드 변수로서 OneHandSword 클래스 타입의 변수를 저장해놓고, attack() 메서드를 수행하면  OneHandSword 클래스의 메서드가 실행되어 데미지가 가하는 형태이다.

즉, Character의 인스턴스 생성 시 OneHandSword에 의존성을 가지기게되어, 공격 동작을 담당하는 attack() 메소드 역시 OneHandSword에 의존성을 가지게 되게 된다.

  

![[image 326.png]]

  

```Java
class Character {
final String NAME;
int health;
OneHandSword weapon; // 의존 저수준 객체
Character(String name, int health, OneHandSword weapon) {
    this.NAME = name;
    this.health = health;
    this.weapon = weapon;
}

int attack() {
    return weapon.attack(); // 의존 객체에서 메서드를 실행
}

void chageWeapon(OneHandSword weapon) {
    this.weapon = weapon;
}

void getInfo() {
    System.out.println("이름: " + NAME);
    System.out.println("체력: " + health);
    System.out.println("무기: " + weapon);
}
}
```

  

하지만 무기엔 한손검 타입만 있는 게 아니다.

위에서 살펴봤듯이 양손검, 전투도끼, 망치 타입의 여러 무기들을 장착하게 하려면, 아예 캐릭터 클래스의 클래스 필드 변수 타입을 교체해줘야 한다.

하지만 만약 위 코드가 의존성 역전 원칙을 잘 지켰다면 고민할 필요가 없는 문제다.

위 코드의 문제는 **이미 완전하게 구현된 하위 모듈을 의존**하고 있다는 점이다.

즉, 구체 모듈을 의존하는 것이 아닌 추상적인 고수준 모듈을 의존하도록 리팩토링 하면 된다.

우선 모든 무기들을 포함할 수 있는 고수준 모듈인 Weaponable 인터페이스를 생성한다.

그리고 모든 공격 가능한 무기 객체는 이 인터페이스를 implements 하게 한다.

  

![[image 327.png]]

  

  

```Java
// 고수준 모듈
interface Weaponable {
int attack();
}

class OneHandSword implements Weaponable {
final String NAME;
final int DAMAGE;
OneHandSword(String name, int damage) {
    NAME = name;
    DAMAGE = damage;
}

public int attack() {
    return DAMAGE;
}
}

class TwoHandSword implements Weaponable {
// ...
}

class BatteAxe implements Weaponable {
// ...
}

class WarHammer implements Weaponable {
// ...
}
```

그리고 Character 클래스의 기존의 OneHandSword 타입의 필드 변수를 좀 더 고수준 모듈인 Weaponable 인터페이스 타입으로 변경한다.

게임 시스템 내부적으로 모든 공격 가능한 무기는 Weaponable 을 구현하기로 가정했으므로, 공격 가능한 모든 무기를 할당 받을 수 있게 된 것이다.

  

```Java
class Character {
final String NAME;
int health;
Weaponable weapon; // 의존을 고수준의 모듈로

Character(String name, int health, Weaponable weapon) {
    this.NAME = name;
    this.health = health;
    this.weapon = weapon;
}

int attack() {
    return weapon.attack();
}

void chageWeapon(Weaponable weapon) {
    this.weapon = weapon;
}

void getInfo() {
    System.out.println("이름: " + NAME);
    System.out.println("체력: " + health);
    System.out.println("무기: " + weapon);
}
}
```

어찌보면 이러한 DIP 원칙을 따름으로써, 무기의 변경에 따라 Character의 코드를 변경할 필요가 없고 또다른 타입의 무기 확장에도 무리가 없으니 OCP 원칙 또한 준수한 것이라고 볼수도 있다.


**출처 : https://inpa.tistory.com/
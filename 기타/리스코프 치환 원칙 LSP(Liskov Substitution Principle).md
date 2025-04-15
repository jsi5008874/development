  

리스코프 치환 원칙은 1988년 바바라 리스코프(Barbara Liskov)가 올바른 상속 관계의 특징을 정의하기 위해 발표한 것으로, **서브 타입은 언제나 기반 타입으로 교체**할 수 있어야 한다는 것을 뜻한다.

교체할 수 있다는 말은, 자식 클래스는 최소한 자신의 부모 클래스에서 가능한 행위는 수행이 보장되어야 한다는 의미이다.

즉, 부모 클래스의 인스턴스를 사용하는 위치에 자식 클래스의 인스턴스를 대신 사용했을 때 코드가 원래 의도대로 작동해야 한다는 의미이다.

  

**쉽게 표하자면 다형성의 원리를 뜻하는 것이고 리스코프 치환 원칙은 다형성을 지원하기 위한 원칙**

  

리스코프 치환 원칙의 핵심은 **부모 클래스의 행동 규약을 자식 클래스가 위반**하면 안 된다는 것이다.

**행동 규약**을 위반한다는 것은 자식 클래스가 **오버라이딩**을 할 때, 잘못되게 재정의하면 리스코프 치환 원칙을 위배할 수 있다는 의미

  

자식 클래스가 오버라이딩을 잘못하는 경우는 크게 두 가지로 나뉜다.

첫번째는 자식 클래스가 부모 클래스의 메소드 시그니처를 자기 멋대로 변경하거나, 두번째는 자식 클래스가 부모 클래스의 의도와 다르게 메소드를 오버라이딩 하는 경우가 있다

  

### **자식의 잘못된 메소드 오버로딩**

  

  

```Plain
class Animal {
int speed = 100;
int go(int distance) {
    return speed * distance;
}
}

class Eagle extends Animal {
String go(int distance, boolean flying) {
if (flying)
return distance + "만큼 날아서 갔습니다.";
else
return distance + "만큼 걸어서 갔습니다.";
}
}

public class Main {
public static void main(String[] args) {
Animal eagle = new Eagle();
eagle.go(10, true);
}
}
```

Animal 클래스를 상속하는 Eagle 자식 클래스가 부모 클래스의 ~~go()~~ 메소드를 자기 멋대로 코드를 재사용 한답 시고 메소드 타입을 바꾸고 매개변수 갯수도 바꿔 버렸다.

한마디로 어느 메소드를 오버로딩을 부모가 아닌 자식 클래스에서 해버렸기 때문에 발생한 LSP 위반 원칙

  

### **부모의 의도와 다르게 메소드 오버라이딩**

  

```Plain
class NaturalType {
String type;
NaturalType(Animal animal) {
// 생성자로 동물 이름이 들어오면, 정규표현식으로 매칭된 동물 타입을 설정한다.
if(animal instanceof Cat) {
type = "포유류";
} else {
// ...
}
}
String print() {
    return "이 동물의 종류는 " + type + " 입니다.";
}
}
```

```Plain
class Animal {
NaturalType getType() {
    NaturalType n = new NaturalType(this);
    return n;
}
}

class Cat extends Animal {
}
```

먼저 Animal 클래스에 확장되는 동물들(Cat, Dog, Lion ...등)을 다형성을 이용하여 업캐스팅으로 인스턴스화 해주고, getType() 메서드를 통해 NautralType 객체 인스턴스를 만들어 NautralType의 print() 메서드를 출력하여 값을 얻는 형태

  

```Java
public class Main {
public static void main(String[] args) {
Animal cat = new Cat();
String result = cat.getType().print();
System.out.println(result); // "이 동물의 종류는 포유류 입니다."
}
}
```

  

```Java
class Cat extends Animal {
@Override
NaturalType getType() {
    return null;
}

String getName() {
    return "이 동물의 종류는 포유류 입니다.";
}
```

그런데 협업하는 다른 개발자가 이런식으로 구성하면 뭔가 번거로울 것 같아, 자기 멋대로 자식 클래스에 부모 메서드인 getType() 의 반환값을 null로 오버라이딩 설정하여 메서드를 사용하지 못하게 설정하고, 대신 getName() 이라는 메서드를 만들어 한번에 출력하도록 설정한 것이다.  
  
이렇게 바꾼다면 물론 오류가 발생한다.  

  

이것이 리스코프 치환 원칙의 중요 포인트다.

자식 클래스로 부모 클래스의 내용을 상속하는데, 기존 코드에서 보장하던 조건을 수정하거나 적용시키지 않아서, 기존 부모 클래스를 사용하는 코드에서 예상하지 않은 오류를 발생시킨 것이다.

만일 컴파일 단에서 오류를 체크해주면 좋을텐데, 코드 구성상 문제가 없기 때문에 이렇게 예측하지 못한 에러가 발생한 것이다.

**따라서 사전에 약속한 기획대로 구현하고, 상속 시 부모에서 구현한 원칙을 따라야 한다가 이 원칙의 핵심이다.**

  

  

### **LSP 원칙 적용 주의점**

결국 리스코프 치환 원칙이란, 다형성의 특징을 이용하기 위해 상위 클래스 타입으로 객체를 선언하여 하위 클래스의 인스턴스를 받으면, 업캐스팅된 상태에서 부모의 메서드를 사용해도 동작이 의도대로만 흘러가도록 구성하면 되는 것이다.

그리고 LSP 원칙의 핵심은 상속(Inheritance)이다.

그런데 주의할 점은, 객체 지향 프로그래밍에서 상속은 기반 클래스와 서브 클래스 사이에 **IS-A 관계**가 있을 경우로만 **제한** 되어야 한다.

그 외의 경우에는 **[합성(composition)Visit Website](https://inpa.tistory.com/entry/OOP-%F0%9F%92%A0-%EA%B0%9D%EC%B2%B4-%EC%A7%80%ED%96%A5%EC%9D%98-%EC%83%81%EC%86%8D-%EB%AC%B8%EC%A0%9C%EC%A0%90%EA%B3%BC-%ED%95%A9%EC%84%B1Composition-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0)**을 이용하도록 권고되어 있다.

따라서 다형성을 이용하고 싶다면 ~~extends~~ 대신 인터페이스로 ~~implements~~하여 인터페이스 타입으로 사용하기를 권하며, 상위 클래스의 기능을 이용하거나 재사용을 하고 싶다면 상속(inheritnace) 보단 합성(composition)으로 구성하기를 권장한다.


**출처 : https://inpa.tistory.com/
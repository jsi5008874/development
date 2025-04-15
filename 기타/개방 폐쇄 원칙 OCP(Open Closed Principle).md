  

**기존의 코드를 변경하지 않으면서, 기능을 추가할 수 있도록 설계**가 되어야 한다는 원칙  
**확장에 대해서는 개방적(open)**이고, **수정에 대해서는 폐쇄적(closed)**이어야 한다는 의미로 정의  
  
  

**[ 확장에 열려있다 ]**

- 모듈의 확장성을 보장하는 것을 의미한다.
- 새로운 변경 사항이 발생했을 때 유연하게 코드를 추가함으로써 애플리케이션의 기능을 큰 힘을 들이지 않고 확장할 수 있다.

**[ 변경에 닫혀있다 ]**

- 객체를 직접적으로 수정하는건 제한해야 한다는 것을 의미한다.
- 새로운 변경 사항이 발생했을 때 객체를 직접적으로 수정해야 한다면 새로운 변경사항에 대해 유연하게 대응할 수 없는 애플리케이션이라고 말한다.
- 이는 유지보수의 비용 증가로 이어지는 매우 안좋은 예시이다.
- 따라서 객체를 직접 수정하지 않고도 변경사항을 적용할 수 있도록 설계해야 한다. 그래서 변경에 닫혀있다고 표현한 것이다.

  

### 예제

  

  

```Plain
class Animal {
String type;
Animal(String type) {
	this.type = type;
}
}

// 동물 타입을 받아 각 동물에 맞춰 울음소리를 내게 하는 클래스 모듈
class HelloAnimal {
void hello(Animal animal) {
if(animal.type.equals("Cat")) {
System.out.println("냐옹");
} else if(animal.type.equals("Dog")) {
System.out.println("멍멍");
}
}
}

public class Main {
public static void main(String[] args) {
HelloAnimal hello = new HelloAnimal();
    Animal cat = new Animal("Cat");
    Animal dog = new Animal("Dog");

    hello.hello(cat); // 냐옹
    hello.hello(dog); // 멍멍
}
}
```

  

동작 자체는 문제가 없지만 문제는 **기능 추가** 이다.

만일 '고양이'와 '개' 외에 '양'이나 '사자'를 추가하게 된다면 어떻게 될까?

당연히 HelloAnimal 클래스를 수정해주어야 한다. 각 객체의 필드 변수에 맞게 if문을 분기하여 구성해줘야 한다.

  

OCP의 규칙

1. 먼저 변경(확장)될 것과 변하지 않을 것을 엄격히 구분한다.
2. 이 두 모듈이 만나는 지점에 추상화(추상클래스 or 인터페이스)를 정의한다.
3. 구현체에 의존하기보다 정의한 추상화에 의존하도록 코드를 작성 한다.

  

![[image 321.png]]

  

  

```Plain
 // 추상화
abstract class Animal {
abstract void speak();
}

class Cat extends Animal { // 상속
void speak() {
System.out.println("냐옹");
}
}

class Dog extends Animal { // 상속
void speak() {
System.out.println("멍멍");
}
}

class HelloAnimal {
void hello(Animal animal) {
animal.speak();
}
}

public class Main {
public static void main(String[] args) {
HelloAnimal hello = new HelloAnimal();  
    Animal cat = new Cat();
    Animal dog = new Dog();

    hello.hello(cat); // 냐옹
    hello.hello(dog); // 멍멍
}
}
```

이처럼 추상화를 통해 설계를 하면 기능 추가가 되었음에도 코드 수정 없이 확장 가능


**출처 : https://inpa.tistory.com/
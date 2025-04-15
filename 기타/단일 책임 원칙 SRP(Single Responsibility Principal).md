  

객체는 단 하나의 책임만 가져야 한다는 원칙

책임은 **하나의 기능 담당**으로 생각하면 쉽다

  

### 객체가 하나의 책임을 가져야 하는 이유

  

만약 객체 A가 여러 기능을 가지고 있는 상태라면

다른 객체들과 강하게 결합될 가능성이 높다.

이 뜻은 객체 A를 수정했을 때 다른 객체들도 수정을 해야 하는 가능성이 높다는 것이다.

**즉 유지보수 측면에서 아주 안좋은 결과를 가져온다**

책임이 이것저것 포함된 클래스는 한 책임의 변경에서 다른 책임의 변경으로의 연쇄작용이 일어 나게 된다.

  

여기서 단일 책임 원칙을 적용한다면, 각 클래스 주제마다 알맞는 책임을 가짐으로서 책임 영역이 확실해지게 된다.

그래서 어떠한 역할에 대해 변경사항이 발생했을때, 변경 영향을 받는 기능만 모아둔 클래스라면 그 책임을 지니고 있는 클래스만 수정해주면 될 일이다.

  

  

예제

  

다음 직원 정보를 담당하는 Employee 클래스에는 4가지의 주요 메서드가 존재한다.

- calculatePay() : 회계팀에서 급여를 계산하는 메서드
- reportHours() : 인사팀에서 근무시간을 계산하는 메서드
- saveDababase() : 기술팀에서 변경된 정보를 DB에 저장하는 메서드
- calculateExtraHour() : 초과 근무 시간을 계산하는 메서드 (회계팀과 인사팀에서 공유하여 사용)

각 메서드들은 각 팀에서 필요할때마다 호출해 사용한다고 가정하자.

아래 예시 코드에서는 calculatePay() 메서드와 reportHours() 메서드에서 초과 근무 시간을 계산하기 위해 calculateExtraHour() 메서드를 공유하여 사용하고 있다.

  

  

```Plain
class Employee {
String name;
String positon;

Employee(String name, String position) {
    this.name = name;
    this.positon = position;
}

// * 초과 근무 시간을 계산하는 메서드 (두 팀에서 공유하여 사용)
void calculateExtraHour() {
    // ...
}

// * 급여를 계산하는 메서드 (회계팀에서 사용)
void calculatePay() {
    // ...
    this.calculateExtraHour();
    // ...
}

// * 근무시간을 계산하는 메서드 (인사팀에서 사용)
void reportHours() {
    // ...
    this.calculateExtraHour();
    // ...
}

// * 변경된 정보를 DB에 저장하는 메서드 (기술팀에서 사용)
void saveDababase() {
    // ...
}
```

  

그런데 회계팀에서 급여를 계산하는 기존의 방식을 새로 변경하여, 코드에서 초과 근무 시간을 계산하는 메서드 calculateExtraHour() 의 알고리즘 업데이트가 필요해졌다.

그래서 개발팀에서 회계팀의 요청에 따라 calculateExtraHour() 메서드를 변경했는데, 변경에 의한 파급 효과로 인해 수정 내용이 의도치 않게 인사팀에서 사용하는 reportHours() 메서드에도 영향을 주게 되어버린다. (공유해서 사용하니까)

그리고 인사팀에서는 이러한 변경 사실을 알지 못하고 메서드 반환 결과값 데이터가 잘못되었다며 개발팀에 새로운 요청을 보내게 될 것이다.

바로 이러한 상황이 SRP에 위배되는 상황인 것이다.

Employee 클래스에서 **회계팀, 인사팀, 기술팀 이렇게 3개의 액터**에 대한 **책임**을 한꺼번에 가지고 있기 때문이다.

  

예제를 SRP원칙을 적용해 변형한 코드는 다음과 같이 된다.

[![](https://blog.kakaocdn.net/dn/daOzyK/btrOEg7usGg/TOiI2oTu7AkBQisHvCHDs0/img.png)](https://blog.kakaocdn.net/dn/daOzyK/btrOEg7usGg/TOiI2oTu7AkBQisHvCHDs0/img.png)

상속 관계를 나타낸 다이어그램이 아니라 사용 관계를 나타낸 다이어그램이다

  

  

```Plain
// * 통합 사용 클래스
class EmployeeFacade {
private String name;
private String positon;

EmployeeFacade(String name, String position) {
    this.name = name;
    this.positon = position;
}

// * 급여를 계산하는 메서드 (회계팀 클래스를 불러와 에서 사용)
void calculatePay() {
    // ...
    new PayCalculator().calculatePay();
    // ...
}

// * 근무시간을 계산하는 메서드 (인사팀 클래스를 불러와 에서 사용)
void reportHours() {
    // ...
    new HourReporter().reportHours();
    // ...
}

// * 변경된 정보를 DB에 저장하는 메서드 (기술팀 클래스를 불러와 에서 사용)
void EmployeeSaver() {
    new EmployeeSaver().saveDatabase();
}


}

// * 회계팀에서 사용되는 전용 클래스
class PayCalculator {
// * 초과 근무 시간을 계산하는 메서드
void calculateExtraHour() {
// ...
}
void calculatePay() {
// ...
this.calculateExtraHour();
// ...
}
}

// * 인사팀에서 사용되는 전용 클래스
class HourReporter {
// * 초과 근무 시간을 계산하는 메서드
void calculateExtraHour() {
// ...
}
void reportHours() {
// ...
this.calculateExtraHour();
// ...
}
}

// * 기술팀에서 사용되는 전용 클래스
class EmployeeSaver {
void saveDatabase() {
// ...
}
}
```

  

단인 책임의 적용은 어렵지 않다. 각 책임(기능 담당)에 맞게 클래스를 분리하여 구성하면 끝이다.

회계팀, 인사팀, 기술팀의 기능 담당은 PayCaculator, HourReporter, EmployeeSaver 각기 클래스로 분리하고, 이를 통합적으로 사용하는 클래스인 EmployeeFacade 클래스를 만든다.

EmployeeFacade 클래스의 메서드에는 사실상 아무런 로직이 들어있지 않게 된다. 있는 코드라 봤자 **생성자로 인스턴스를 생성하고 각 클래스의 메서드를 사용하는 역할**만 할 뿐이다.

이렇게 되면 EmployeeFacade 클래스는 **어떠한 액터도 담당하지 않게 된다.**

만일 변경 사항이 생겨도, 각각의 분리된 클래스에서만 수정하면 되기 때문에 EmployeeFacade 클래스는 냅둬도 기능을 이용하는데 있어 아무런 문제가 생기지 않는다.

예를 들어 회계팀에서 초과 근무 시간 계산 방법이 변경되었다고 하면, PayCalculator 클래스의 calculateExtraHour() 메서드를 수정하면 될 일이다. 그리고 이 변경사항이 HourReporter 클래스에는 전혀 영향을 주지 않게 된다.

  

  

SRP 범위에 대한 고찰

프로그램을 설계 할때 객체에 얼만큼의 책임을 넣어야 단일 책임 원칙을 위배하지 않는지 범위 기준은 개발자 스스로 정해야 한다.

그리고 실무의 프로세스는 매우 복잡 다양하고 변경 또한 빈번하기 때문에 경험이 많지 않거나 업무 이해가 부족하면 자기도 모르게 SRP원리에서 멀어져 버리게 될 수도 있다.

따라서 평소에 많은 연습과 경험이 필요한 원칙

  

주의점

**1. 클래스명은 책임의 소재를 알수있게 작명**

클래스가 하나의 책임을 가지고 있다는 것을 나타내기 위해, **클래스명**을 어떠한 기능을 담당하는지 알수 있게 작명하는 것이 좋다.

즉, 각 클래스는 하나의 개념을 나타내게 구성하는 것이다.

**2. 책임을 분리할때 항상 결합도, 응집도 따져가며**

SRP 원칙을 따른다고 해서 무턱대로 책임을 아무생각 없이 분리하면 안되고, 항상 결합도와 응집도를 따져가며 구성해야 한다.

응집도란 한 프로그램 요소가 얼마나 뭉쳐있는가를 나타내는 척도이고, 결합도는 프로그램 구성 요소들 사이가 얼마나 의존적인지를 나타내는 척도이다.

좋은 프로그램이란 **응집도를 높게, 결합도는 낮게** 설계하는 것을 말한다.

따라서 여러가지 책임으로 나눌때는 각 책임간의 결합도를 최소로 하도록 코드를 구성해야 한다.

하지만 그 반대로 너무 많은 책임 분할로 인하여 책임이 여러군데로 파편화 되어있는 경우에는 **산탄총 수술**로 다시 응집력을 높여주는 작업이 추가로 필요하다.


**출처 : https://inpa.tistory.com/
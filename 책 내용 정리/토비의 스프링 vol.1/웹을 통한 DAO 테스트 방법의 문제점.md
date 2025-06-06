  

웹 프로그램에서 DAO를 테스트한다고 가정했을 때

DAO를 테스트하기 위해서 서비스, 컨트롤러, 뷰 등 모든 레이어의 기능을 만들고 나서야

테스트가 가능하다는 단점이 있다.

**내가 테스트하고 싶은 것은 DAO 클래스인데 DAO의 기능이 잘 작동하는지 확인하기 위해**

**다른 계층의 기능까지 붙어있어야 확인이 가능하다.**

  

또한 테스트 중 오류가 생겼을 때 해당 오류가 DAO에서 발생한건지 다른 계층에서 발생한건지

파악하는데도 시간이 소요된다.

  

이런 문제를 피해서 효율적인 테스트를 하기 위해서 어떤 방법이 있을까?

  

테스트하고자 하는 대상이 명확하다면 그 대상에만 집중해서 테스트를 하면 된다.

### 작은 단위 테스트

테스트에도 관심사의 분리를 적용하는 것이다.

UserDaoTest에서는 웹 인터페이스, 서비스 오브젝트 없이 UserDao와 DB 연결 방법에 관심을 두고

해당 기능만 테스트를 했다.

**이처럼 작은 단위의 코드에 대해 테스트를 수행하는 것을 단위 테스트(Unit Test)라고 한다.**

  

일반적으로 단위는 작을수록 좋고 단위를 넘어서는 다른 코드들은 신경 쓰지도 참여하지 않는다.

하나의 관심사에 집중해서 테스트를 하는 것이다.

  

단위 테스트를 하는 이유는 개발자가 설계한 코드가 의도대호 동작하는지 스스로 확인하기 위함이다

만약 여러 기능을 개발한 후 큰 규모의 테스트를 한다면 여러 오류가 발생할 가능성이 있고 디버깅에

그만큼 시간을 들여야한다.

또한 개발자가 개발 직후 단위 테스트를 하면서 디버깅을 하는 것과 여러 기능을 개발 후 한번에 테스트를 하는것에는 분명히 효율차이가 발생한다.

  

### 자동수행 테스트 코드

UserDaoTest의 특징은 테스트할 데이터가 코드를 통해 제공되고 테스트 작업 역시 코드를 통해 자동으로 실행된다.

  

테스트에 쓰인 데이터인 USer에 대한 정보는 코드로 setName, setAge등으로 제공했고

IDE에서 run 버튼만 누르면 자동으로 DB에 insert 되도록 구현했다.

  

이처럼 테스트는 자동으로 수행되도록 만들어야한다.

웹 화면에 폼을 띄우고 매번 User의 등록 값을 개발자가 스스로 입력하고 버튼을 누르는 등

테스트에 이러한 작업이 반복된다면 쉽게 지칠 것이다.

  

### 테스트 검증 자동화

테스트가 자동으로 수행되더라도 테스트에 대한 결과는 개발자가 직접 확인해야한다.

UserDaoTest를 예시로 보면 데이터를 insert하고 해당 데이터를 println(get())으로 출력해서 확인한다

이처럼 검증은 개발자가 직접하는데 문제는 이런 단위 테스트가 수십, 수백개가 된다면, 테스트 결과가 한 글자 차이를 찾아내야한다면 개발자에게 엄청난 피로도가 생긴다.

  

이런 문제를 해결하기 위해 테스트 검증을 자동화한다.

  

ex)

```Java
if(!user.getName()equals(user2.getName()){
   system.out.println("테스트 실패 (name)");
}
else if{!user.getPassword().equals(user2.getPassword())){
      system.out.println("테스트 실패 (password)");
}
else{
       system.out.println("조회테스트 성공");
       }
```

이런식으로 직접 한글자씩 확인하는 것 보다 각 데이터를 비교하면서 테스트 결과를 출력해주는게

더 효율적인 방법이다.

  

이렇게 자동화된 테스트를 만들어 놓으면 큰 변화가 일어나도 UserDao가 전과 같이 작동하는지

이 테스트 한 번이면 충분하다.

여기서 더 나아가 만들어진 코드의 기능을 모두 점검할 수 있는 포괄적인 테스트를 만들면

개발한 애플리케이션은 어떤 과감한 수정을 하더라도 테스트를 돌려보고 나면 안심이 된다.

이는 개발자가 자신의 코드에 자신감을 가질 수 있으며 실무에서도 가장 확실하고 빠르게 기능에 대해 테스트하고 업무를 수행하게 만들어준다.
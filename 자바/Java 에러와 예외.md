  

**컴파일 에러(Compile-Time Error) : 컴파일 시 발생하는 에러**

**런타임 에러(Runtime Error) : 실행 시 발생하는 에러**

**논리적 에러(Logical Error) : 실행은 되지만 개발 의도와 다르게 동작하는 에러**

  

### 논리 에러(Logical Error)

프로그램은 이상 없이 실행되지만 결과가 예상과 다르게 동작하는 에러

게임의 버그와 같은 개념

ex) 몹의 체력이 0이 되면 죽어야하는데 죽지 않는 버그 같은 경우

  

**논리적 오류는 프로그램이 실행은 되기 때문에 에러 메시지를 알려주지 않는다.**

**따라서 개발자가 프로그램의 전반적인 코드를 체크해봐야한다.**

  

### 컴파일 에러(Compilation Error)

컴파일 단계에서 발생하는 오류로 에러 메시지를 출력해준다.

대표적인 원인은 문법 구문 오류(Syntax error)가 있다.

ex) 맞춤법, 존재하지 않는 변수명 등 IDE에서 빨간줄로 잘못 작성된 부분을 명시해주는 것

  

컴파일 에러는 대응이 가능하다.

프로그램이 실행되기 전에 발생하는 오류로 컴파일 에러가 발생하면 프로그램 실행이 안되기 때문에

충분히 분석하고 오류를 해결할 수 있다.

  

### 런타임 에러(Runtime Error)

프로그램 실행 중 발생하는 오류로 기계적 결함, 개발 시 설계 미숙으로 발생할 수 있다.

  

런타임 에러는 프로그램이 실행 중 강제로 종료될 수 있기 때문에 개발 설계 시 발생 가능한

여러 가지 경우의 수를 염두에 두고 설계해야한다.

  

**오류(Error)와 예외(Exception)**

![[image 311.png]]

자바에서는 런타임 오류를 에러와 예외로 구분한다.

  

에러 : 수습할 수 없는 심각한 오류

예외 : 수습될 수 있는 미약한 오류

  

에러는 메모리 부족(Out of Memory Error), 스택 오버 플로우(Stack over flow error) 같은

발생하면 복구할 수 없는 심각한 오류이고 JVM 실행에 문제가 생긴것 이기 때문에 개발자가 대처할 방법이 없다.

  

예외는 보통 프로그램 상 알고리즘 오류로 개발자가 설계한 로직의 문제이거나 사용자의 영향에 의해 발생한다.

  

보통 설계 시 예외 처리 문법(try-catch)을 사용하여 대응 코드를 미리 작성한다.

  

**예외 클래스 계층 구조**

자바에서는 에러와 예외를 클래스로 구현하여 처리

JVM은 프로그램 실행 중 예외가 발생하면 해당 예외 클래스로 객체를 생성하여 예외 처리 코드에서

예외 객체를 이용할 수 있도록 해준다.

  

![[image 312.png]]

  

Throwable 클래스 : 오류와 예외의 최상위 클래스, getMessage()와 printStackTrace() 메서드가

해당 클래스에 속해 있고 하위 클래스들도 모두 사용할 수 있다.

  

**Exception의 종류**

![[image 313.png]]

exception에서도 컴파일 예외(파란색)와 런타임 예외(빨간색)로 구분한다.

(RuntimeException 클래스의 자식 클래스를 제외하면 컴파일 예외)

  

### Runtime Exception 클래스 종류

![[image 314.png]]

- **ArrayIndexOutOfBoundsException :** 배열의 범위를 넘어선 인덱스를 참조할 때 발생하는 에러
- **ArithmeticException :** 정수를 0으로 나눌 때 발생하는 에러
- **NullPointException :** null 객체에 접근해서 method를 호출하는 경우 발생하는 에러 (객체가 없는 상태에서 객체를 사용하려 했으니)
- **NumberFormatException :** 정수가 아닌 문자열을 정수로 변환할 때 예외 발생
- **ClassCastException**

-타입 변환은 상위 클래스와 하위 클래스간의 상속 관계 이거나 혹은 구현 클래스와 인터페이스

일 때만 가능

-상속, 구현 관계 아니면 클래스는 다른 클래스로 타입을 변환할 수 없는데, 이 규칙을 무시하고

억지로 타입을 변환시킬경우 발생하는 에러이다.

- **InputMismatchException :** 의도치 않는 입력 오류 시 발생하는 예외

  

### Compile Exception 종류

- **IOException**

-컴퓨터 프로그램이 실행될 때 언제 어떤 문제가 발생할지 모르는 일이기 때문에, 컴퓨터와 상호

소통 하는 I/O(입력과 출력)에 관해서는 발생할 수 있는 예외에 대해서 까다롭게 규정하고 있다.

-그래서 입력과 출력을 다루는 메서드에 예외처리(IOException)가 없다면 컴파일 에러가 발생

- **FileNotFoundException :** 파일에 접근하려고 하는데 파일을 찾지 못했을 때 발생하는 에러

  

  

### Checked Exception / Unchecked Exception

  

**Checked Exception = 컴파일 예외**

**Unchecked Exception = 런타임 예외**

  

이렇게 나눈 기준은 코드를 작성할 때 **예외 처리 동작 필수 지정 여부**이다.

즉 반드시 예외 처리를 해야 하는지 아닌지의 차이

![[image 315.png]]

  

Checked Exception은 반드시 try-catch문을 사용하거나 throws를 사용해야 컴파일이 진행된다.

그렇지 않으면 컴파일이 진행되지 않아서 프로그램 실행이 안됨

  

반면 unchecked exception은 예외처리가 필수는 아니므로 프로그램 실행은 되지만 예외 처리를 하지 않으면 예외 발생 시 처리 불가

  

```Java
// try - catch 로 예외처리
public static void fileOpen() {
// 파일을 열고 쓰고 닫는 아주 단순한 로직이어도 이에 대한 예외는 checked exception으로 분류 되기 때문에 반드시 try - catch로 감싸주어야 한다.
try {
        FileWriter file = new FileWriter("data.txt");
    	file.write("Hello World");
    	file.close();
    } catch(IOException e) {
    	e.printStackTrace();
    }
}
) {
```

```Java
// throws 로 예외처리
public static void fileOpen() throws IOException {
// 파일을 열고 쓰고 닫는 아주 단순한 로직이어도 이에 대한 예외는 checked exception으로 분류 되기 때문에 반드시 try - catch로 감싸주어야 한다.
FileWriter file = new FileWriter("data.txt");
    file.write("Hello World");
    file.close();
}
```

  

### Checked를 Unchecked 예외로 변환하기

  

checked > unchecked로 변환하는 이유는 처음 자바가 개발 될 당시와 현재의 컴퓨터 환경이

많이 바뀌었기 때문이다.

  

실제로 런타임 예외로 처리해도 될 것들이 아직도 checked로 등록되어 강제로 try-catch문을 사용해야하는 불편함이 있고 로직상 Runtime Exception으로 밖에 할 수 없는 경우가 있어서

추가된 기법이다.

  

해당 기법은 연결된 예외(Chained Exception)이라 한다.

  

```Java
class MyCheckedException extends Exception { ... } // checked excpetion

public class Main {
    public static void main(String[] args) {
            install();
    }

    public static void install() {
        throw new RuntimeException(new IOException("설치할 공간이 부족합니다."));
        // Checked 예외인 IOException을 Unchecked 예외인 RuntimeException으로 감싸 Unchecked 예외로 변신 시킨다
    }
}
```

  

### 연결된 예외(Chained Exception)

한 예외가 다른 예외를 발생시킬 수 있는 기능

  

클래스를 상속하여 다형성을 이용해 부모 클래스 타입으로 다뤄온 것 처럼

예외도 부모 예외로 감싸서 활용할 수 있다.

  

예외 A가 발생햇다면 예외 B로 감싸서 throw 하는 식으로 던진다.

**A가 B를 발생시켰다면 A를 B의 원인예외(Cause Exception)이라 한다.**

  

Exception 클래스가 상속하는 Throwable 클래스에는 getMessage(), printStackTrace()외에

연결된 예외를 가능하게 해주는 메서드가 있다.

  

Throwable initCause(Throwable cause) : 지정한 예외를 원인 예외로 등록

Throwable getCause() : 원인 예외를 반환

  

```Java
class InstallException extends Exception { ... }
class SpaceException extends Exception { ... }
class MemoryException extends Exception { ... }

public class Main {
    public static void main(String[] args) {
        try {
            install();
        } catch (InstallException e) {
            System.out.println("원인 예외 : " + e.getCause()); // 원인 예외 출력
            e.printStackTrace();
        }
    }

    public static void install() throws InstallException {
        try {
            throw new SpaceException("설치할 공간이 부족합니다."); // SpaceException 발생

        } catch (SpaceException e) {
            InstallException ie = new InstallException("설치중 예외발생"); // 예외 생성
            ie.initCause(e); // InstallException의 원인 예외를 SpaceException으로 지정
            throw ie; // InstallException을 발생시켜 상위 메서드로 throws 된다.
        } catch (MemoryException e) {
            // ...
        }
    }
}
```

1. startInstall() 메서드에서 SpaceException 이라는 예외가 발생
2. catch 문에서 InstallExceptoin 예외 클래스를 새로 생성
3. 그리고 InstallException 객체의 메서드 initCause()를 이용해 SpaceException 타입의 객체를 넣어 실행
4. 그러면 SpaceException 예외는 InstallException 예외에 포함되게 된다. (원인 예외)
5. InstallException 예외 객체를 밖으로 throws 한다.
6. 메인메서드에서 InstallException 예외를 catch하고 getCause() 메서드를 통해 원인 예외 로그를 출력한다

  

![[image 316.png]]

에러에 대응을 하기위해서는 단순한 결과보다는 단계별로 어떠한 에러에 의해 에러가 발생했다는

과정을 보여주는 것이 더 분석하기 쉽다.

  

예를 들면 단순히 설치할 공간이 부족합니다 라는 에러만 띄우면 어떠한 원인 동작으로 갑자기 공간이 부족한지 추적을 못한다.

하지만 예외를 감싸서 설치중에 >> 설치할 공간이 부족해 예외 발생으로 에러가 발생한 과정을 볼 수 있다면 코드의 어느 부분을 먼저 찾아봐야할 지 알 수 있다.
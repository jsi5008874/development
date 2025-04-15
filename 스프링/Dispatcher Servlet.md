  

Spring MVC 도입 후 생긴 기능으로

**디스패처 서블릿이란 서블릿 컨테이너의 가장 앞단에서 HTTP 프로토콜로 들어오는 모든 요청을 먼저 받아 적합한 컨트롤러에 위임해주는 프론트 컨트롤러이다.**

  

  

![[image 317.png]]

  

### Dispatcher Servlet의 흐름

  

![[image 318.png]]

1. 처음 클라이언트에서 요청이 오면 **디스패처 서블릿**이 해당 요청을 받는다.

2.**Handler Mapping**을 통해 요청에 알맞은 컨트롤러를 찾아낸다.

핸들러는 컨트롤러 안에서 어떤 요청을 처리할 수 있는 메소드를 핸들러라고 부릅니다.

![[image 319.png]]

3. 찾아낸 컨트롤러를 **Handler Adapter**를 통해 해당 컨트롤러의 메서드를 실행시킨다.

Handler Adapter는 컨트롤러와 dispatcher servlet을 이어주는 역할을 한다.

4. 컨트롤러는 요청을 처리한 뒤 처리한 결과와 해당**뷰 정보(ModelAndView)**를 다시 디스패처 서블릿에게 전달한다.

5. 받은 정보로 디스패처 서블릿은 **View Resolver**를 통해 **View** 파일을 찾는다.

1. view resolver가 view 이름을 실제 html, jsp 등의 파일 경로로 변환
2. 최종적으로 뷰가 HTML에 렌더링되고 사용자에게 응답으로 반환

  

### 여기서 굳이 Handler Mapping과 Handler Adapter로 기능을 분리한 이유는?

그냥 Handler Mapping이 컨트롤러를 찾고 바로 dispatcher servlet이 요청을 컨트롤러에 건내주면

되는거 아닌가? 생각할 수 있는데

  

그 이유는 Controller의 다양한 타입에 맞게 호환성을 가지고 동작하도록 만들기 위함이다.

Controller의 다양한 타입이란

![[image 320.png]]

첫 번째 컨트롤러는 우리가 흔히 알고 있는 Controller

두 번째는 servlet API를 활용한 컨트롤러(MVC 이전의 servlet이라 생각하면된다.)

이렇게 컨트롤러에도 다양한 타입이 있고 컨트롤러마다 호출방법과 요청처리방식이 다르다.

  

따라서 Handler Adapter를 따로 둬서 컨트롤러 타입에 맞는 호출방법으로 매핑을 해주는 것이다.
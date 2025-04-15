  

클라이언트의 요청을 처리하고, 그 결과를 반환하는

Servlet 클래스의 구현 규칙을 지킨 자바 웹 프로그래밍 기술

  

일반적으로 웹서버는 정적컨텐츠(HTML, CSS, 이미지, JS파일 등)만 제공하지만

동적컨텐츠를 제공하기 위해 WAS의 도움을 받는다.

여기서 동적컨텐츠란 DB를 거쳐서 데이터를 가져와서 페이지에 표시해주거나

비즈니스 로직을 거쳐 나온 데이터를 페이지에 표시해주는 것을 말한다.

ex)게시판 : 새로운 글이 생기면 게시판의 목록이 바뀌는 등

  

동적컨텐츠를 만들기 위해 Servlet이 필요함

Servlet Class는 @WebServlet 어노테이션을 사용해서 선언하고

그 안에 동적으로 처리할 메서드를 정의한다.

  

ex) 로그인 서블릿

@WebServlet("/login")  
public class LoginServlet extends HttpServlet {  
protected void doPost(HttpServletRequest request, HttpServletResponse response) throws IOException {  
// 요청 파라미터 가져오기  
String username = request.getParameter("username");  
String password = request.getParameter("password");  

```Plain
    // 간단한 로그인 인증
    if ("admin".equals(username) && "password".equals(password)) {
        // 인증 성공 시 응답
        response.setContentType("text/html");
        response.getWriter().println("<html><body>Welcome, " + username + "!</body></html>");
    } else {
        // 인증 실패 시 401 상태 코드와 에러 메시지 출력
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        response.getWriter().println("Unauthorized: Invalid credentials.");
    }
}
```

}

이런식으로 동적으로 HTTP 요청에 따른 응답을 작성해주는 역할을 한다.

이런 Servlet Class는 Web.xml에서 지정을 해주고 해당 URL에 대한 요청이 들어오면 자동으로

지정된 Servlet Class로 연동되어 응답을 작성한다.

  

**즉 servlet class안에서 비즈니스 로직 처리, DB연결 등 응답에 필요한 모든 일들을 해결했다.**

지금처럼 Controller, service, repository등으로 나누는 계층화된 개념이 없었던 시절이었다.

이로 인해 servlet class의 코드가 너무 난잡해지고 유지보수도 어려워져서 나온게

Spring MVC이다.

  

Spring MVC로 넘어오면서 Controller가 이 역할을 대신하게 되어 따로 @WebServlet으로

Servlet Class를 만들지는 않는다.

  

Spring MVC에서는 Dispatcher Servlet이 역할을 대신하는데

클라이언트의 요청을 처리하고 적절한 컨트롤러에 매핑하는 역할을 한다.

기본적으로 HTTP 요청을 받고, 요청 URL에 따라 알맞은 컨트롤러 메서드를 호출하며, 그 결과를 적절한 뷰로 전달

  

자바 웹기술 역사는

서블릿 → JSP → (서블릿 + JSP) MVC 패턴 → 스트럿츠, 웹워크 -> 스프링 MVC

  

정리하자면

**Spring MVC가 나오기 이전에는 개발자가 URL마다 직접 servlet 클래스를 정의해서**  
**요청에 대한 응답을 해당 URL에 매핑되는 servlet 클래스가 처리하고**  
**이런 servlet들을 servlet container가 관리했었는데**  
**Spring MVC 이후에는 Dispatcher sevlet이라는 개념이 나와서**  
**dispatcher servlet이 모든 요청에 대한 매핑을 컨트롤러로 해주고 servlet container는**  
**servlet class들을 관리하는게 아닌 dispatcher servlet만 관리**  

  

[[Dispatcher Servlet]]
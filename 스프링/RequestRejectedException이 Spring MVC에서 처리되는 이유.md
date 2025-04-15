  

**RequestRejectedException : Spring Security의** `**StrictHttpFirewall**` 또는 관련 필터에서

보안 규칙을 위반하거나 HTTP 요청의 구조가 보안 정책과 맞지 않을 때 발생

  

최근 Slack에서 **RequestRejectedException이 자주 발생했다.**

이유는 naver 링크를 통해 회사 페이지로 접근할 때 http header에 인코딩이 잘못된 문자열이

포함되어 발생하는 예외였다.

  

![[do-messenger_screenshot_2024-12-16_10_31_26.png]]

  

여기서 의문점은 오류로그를 보니 Handler Mapping >> GlobalExceptionHandler순서였다.

그럼 위 의 순서를 봤을 때 Handler Mappimg은 Spring Security 이후의 과정인데  
왜 Spring Security에서 처리되지 않고 MVC 단계(Dispatcher Servlet 이후)로 넘어가서  

예외 처리가 될까? 라는 의문이었다.

  

그 이유는 **책임의 분리(Separation of Concerns)**이다.

Spring Security : **요청의 보안 검증**과 관련된 작업을 담당

Spring MVC : 비즈니스 로직, 예외 처리 등 어플리케이션 전반적인 부분 담당

  

즉 분업의 개념이다.

Spring Secutity는 요청의 보안 검증 관련된 작업만 담당하므로 문제가 발생했다는 사실만 판단하고

구체적인 예외 처리나 응답 메세지는 어플리케이션 계층(Spring MVC)에서 처리하도록 하는 것이다.

  

Spring Security에서 예외 처리를 할 수 있지만 Spring Security와 Spring MVC 간의 예외 처리 경로가 일관 되지 않을 가능성이 있고 Spring MVC가 @ExceptionHandler, @ControllerAdvice로 예외를 처리할 수 있는 도구들이 있기 때문에 Spring MVC로 넘겨서 예외를 처리하는 것이다.

또한 Spring MVC로 예외를 넘김으로써 모든 예외를 한 곳에서 처리하고 관리 할 수 있어서

유지보수성과 예외 처리 로직의 가독성을 높인다.

  

결론 : 일관적인 예외처리와 유지보수의 용이성 때문에 Spring MVC에서 예외를 일괄 처리

  

### 추가사항

하지만 Spring Security에서 발생하는 모든 Exception을 Spring MVC로 넘겨서 처리하는 것은 아니다.

Spring Security는 **인증(Authentication)**과 **인가(Authorization)** 과정에서 발생하는 예외를 주로 자체적으로 처리

  
**예시**

- **AuthenticationException**
    - 발생 상황: 인증 실패 (예: 잘못된 자격 증명).
    - 처리 방식: `**AuthenticationEntryPoint**`를 통해 처리.
        - 예: 로그인 페이지로 리다이렉트하거나, 401 Unauthorized 응답 반환.
- **AccessDeniedException**
    - 발생 상황: 사용자가 권한이 부족한 리소스에 접근하려고 시도.
    - 처리 방식: `**AccessDeniedHandler**`를 통해 처리.
        - 예: 권한 부족 페이지로 리다이렉트하거나, 403 Forbidden 응답 반환.

  

Spring Security는 **보안 컨텍스트 내**에서 발생하는 예외를 주로 처리하며, 보안과 무관한 예외나 처리 범위를 넘어선 예외(Filter Chain 외부에서 발생한 예외)는 Spring MVC로 넘긴다.
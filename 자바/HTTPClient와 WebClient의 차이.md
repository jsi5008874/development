  

HTTPClient(class)는 HTTP 연결 설정과 관련된 클래스로 요청 전송에 필요한 연결 설정, 타임아웃, 보안 설정 등을 관리

  

WebClient는 실제 HTTP 요청 및 응답을 처리

  

따라서 HTTPClient는 WebClient에 주입해 요청을 전송하는데 필요한 설정을 담당하고 WebClient를 통해 요청이 전송된다.
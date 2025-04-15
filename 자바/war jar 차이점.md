  

### jar(Java Archive)

.jar에는 Class 같은 Java 리소스와 속성 파일, 라이브러리 등 이 포함되어 있다.

쉽게 JAVA 어플리케이션이 동작하도록 자바 프로젝트를 압축한 파일

원하는 구조로 구성할 수 있다.

JRE(Java Runtime Environment)만 가지고도 실행 가능

  

### war(Web Application archive)

.war는 servlet, jsp 컨테이너에 배치 할 수 있는 웹 어플리케이션 압축 파일 포맷

jsp, servlet, jar, class, xml, html, javascript 등 웹 어플리케이션이 구동되기 위한 기타 자원을 한 군데에 모아 배포하는데 사용

WEB-INF, META-INF 디렉토리로 사전 정의 된 구조를 사용

또한 war를 실행하려면 tomcat 같은 웹서버 또는 웹 컨테이너(WAS)가 필요

  

WAR 파일도 JAVA의 JAR 옵션( java - jar)을 이용해 생성하는 JAR파일의 일종으로 웹어플리케이션 전체를 패키징하기 위한 JAR파일로 생각하시면 될 것 같습니다.

  

### war가 jar보다 더 큰 개념이다.

jar는 java application만 구동하기 위한 것이고 war는 java application 뿐만 아니라 웹에서 실행되기 위한 리소스들도 패키징한다.
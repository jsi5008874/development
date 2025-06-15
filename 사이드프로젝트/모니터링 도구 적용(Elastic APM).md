---
~
---
### Elastic APM 적용 이유
여러 APM 도구들이 있지만 현재 Elastic stack을 구현해 놓은 상황이라
Elastic APM을 선택했다.

jaeger, zipkin, pinpoint 등 여러 모니터링 도구가 존재하고 각각의 강점이 있었다.

하지만 현재 ELK가 모두 구현되어 있고 kibana의 뛰어난 시각화 기능을 활용할 수 있어서 최종적으로 Elastic APM을 선택했다.


### Elastic APM의 구조

![[Pasted image 20250615123614.png]]

APM의 구성 요소는 크게 두 가지로 나뉜다.

1. APM agent
	spring boot 앱에서 실행되며 앱에서 발생한 데이터를 수집하여 apm server로 전달하는 역할
	- 메소드 실행 시간 측정
	- HTTP 요청/응답 모니터링
	- DB 쿼리 실행 시간 추적
	- JVM 메모리, CPU 사용량 수집
	- 에러/예외 발생 시 스택 트레이스 캡처
	- 수집한 데이터를 APM Server로 전송
2. APM server
	apm agent로 부터 받은 데이터를 처리하여 elasticsearch에 전달하는 역할

**apm agent가 앱이 실행되면서 발생한 정보를 수집 > apm server가 데이터를 가공 하여 elasticsearch에 전달 >**
**kibana에서 조회**

따라서 apm의 구조를 봤을 때 apm agent는 spring boo app에 설정하고 apm sever는 따로 설정해야한다.

## 설정 파일 수정
### docker-compose 설정
```yml
  # APM Server 추가
  apm-server:
    image: docker.elastic.co/apm/apm-server:7.11.1
    container_name: apm-server
    environment:
      - output.elasticsearch.hosts=["elasticsearch:9200"]
      - apm-server.host=0.0.0.0:8200
      - apm-server.frontend.enabled=true
      - apm-server.frontend.rate_limit=100000
      - apm-server.read_timeout=1m
      - apm-server.shutdown_timeout=2m
      - apm-server.write_timeout=1m
      - logging.level=info
    ports:
      - "8200:8200"
    networks:
      - elk
    depends_on:
      - elasticsearch

# Spring environment APM Agent 설정 추가
- ELASTIC_APM_SERVICE_NAME=anonichat-app          # APM에서 표시될 서비스명
- ELASTIC_APM_SERVER_URLS=http://apm-server:8200  # APM 서버 주소
- ELASTIC_APM_APPLICATION_PACKAGES=com.anonichat  # 모니터링할 패키지
- ELASTIC_APM_ENVIRONMENT=docker                  # 환경 구분
- ELASTIC_APM_LOG_LEVEL=INFO                      # 로그 레벨
- ELASTIC_APM_ENABLE_LOG_CORRELATION=true         # 로그 연관성 활성화

#kibana Environment 추가
- xpack.apm.ui.enabled=true # Kibana에서 APM 메뉴 활성화
```

### spring app dockerfile 수정
```
# apm agent jar 추가
RUN curl -o /app/elastic-apm-agent.jar \  
    https://repo1.maven.org/maven2/co/elastic/apm/elastic-apm-agent/1.28.4/elastic-apm-agent-1.28.4.jar

# apm agent 실행 명령어 추가
ENTRYPOINT ["java", "-javaagent:/app/elastic-apm-agent.jar", "-jar", "/app/AnoniChatApp.jar"]
```

### build.gradle 수정
```
# dependency 추가

// Elastic APM Agent (프로그래밍 방식 연결용)
implementation 'co.elastic.apm:apm-agent-attach:1.50.0'

// APM Spring Boot Starter (자동 설정 - Spring Boot 3.x 호환)
implementation 'co.elastic.apm:elastic-apm-spring-boot-starter:1.50.0'

// APM과 로그 연동 (ECS 로그 형식)
implementation 'co.elastic.logging:logback-ecs-encoder:1.6.0'
```

### application.properties 수정
```
# APM 설정
elastic.apm.service-name=${ELASTIC_APM_SERVICE_NAME:anonichat-app}
elastic.apm.server-urls=${ELASTIC_APM_SERVER_URLS:http://localhost:8200}
elastic.apm.application-packages=${ELASTIC_APM_APPLICATION_PACKAGES:Anoni}
elastic.apm.environment=${ELASTIC_APM_ENVIRONMENT:local}
elastic.apm.enabled=${ELASTIC_APM_ENABLED:true}
elastic.apm.transaction-sample-rate=${ELASTIC_APM_SAMPLE_RATE:1.0}
```

### logback-spring.xml 수정
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- 콘솔 출력용 appender (APM 트레이스 ID 포함) -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <!-- APM 트레이스 정보 추가: trace.id, transaction.id -->
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} [trace.id=%X{trace.id:-} transaction.id=%X{transaction.id:-}] - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- ECS 형식 appender (Elastic Stack 최적화, APM 완벽 연동) -->
    <appender name="ECS_LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <destination>logstash:5000</destination>  <!-- 기존 포트 5000 사용 -->
        <encoder class="co.elastic.logging.logback.EcsEncoder">
            <!-- APM 서비스명 설정 -->
            <serviceName>${elastic.apm.service-name:-anonichat-app}</serviceName>
            <!-- APM 트레이스 정보 자동 포함 -->
            <includeMarkers>true</includeMarkers>
            <includeMdc>true</includeMdc>
        </encoder>
        <keepAliveDuration>5 minutes</keepAliveDuration>
    </appender>

    <!-- 주석 처리된 추가 appender (필요시 사용) -->
    <!--
    <appender name="CUSTOM_LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <destination>logstash:5001</destination>
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
            <includeContext>true</includeContext>
            <includeMdc>true</includeMdc>
            <customFields>{"service":"${elastic.apm.service-name:-anonichat-app}"}</customFields>
        </encoder>
        <keepAliveDuration>5 minutes</keepAliveDuration>
    </appender>
    -->

    <!-- 전체 시스템 로그: 콘솔 + ECS 형식만 사용 -->
    <root level="info">
        <appender-ref ref="CONSOLE" />
        <appender-ref ref="ECS_LOGSTASH" />   <!-- ECS 형식만 사용 (APM 최적화) -->
    </root>

    <!-- 특정 logger 설정 (MainServiceLogger 사용시) -->
    <logger name="MainServiceLogger" level="debug" additivity="false">
        <appender-ref ref="ECS_LOGSTASH" />   <!-- ECS 형식만 사용 -->
        <appender-ref ref="CONSOLE" />
    </logger>

    <!-- APM 관련 로그 레벨 설정 -->
    <logger name="co.elastic.apm" level="INFO" additivity="false">
        <appender-ref ref="CONSOLE" />
    </logger>

    <!-- Spring Boot 및 애플리케이션 로그 레벨 -->
    <logger name="Anoni" level="DEBUG"/>
    <logger name="org.springframework.web" level="DEBUG"/>
    <logger name="org.hibernate.SQL" level="DEBUG"/>

    <!-- 추가 커스텀 logger 예시 (주석 처리)
    <logger name="CustomServiceLogger" level="debug" additivity="false">
        <appender-ref ref="CUSTOM_LOGSTASH" />
        <appender-ref ref="CONSOLE" />
    </logger>
    -->
</configuration>
```
변경점 : 로그의 데이터 형식을  json 형식 > ecs 형식으로 변경

**ECS(Elastic Common Schema)란?**
Elasticsearch에서 정의한 표준화된 로그 데이터 형식

```json
# 일반 json 형식
{
  "timestamp": "2025-06-15T14:30:15.123Z",
  "level": "INFO",
  "message": "사용자 로그인",
  "userId": "user123",
  "ip": "192.168.1.100",
  "userAgent": "Chrome/91.0"
}

# ECS 형식
{
  "@timestamp": "2025-06-15T14:30:15.123Z",
  "log": {
    "level": "INFO"
  },
  "message": "사용자 로그인",
  "user": {
    "id": "user123"
  },
  "client": {
    "ip": "192.168.1.100"
  },
  "user_agent": {
    "original": "Chrome/91.0"
  },
  "service": {
    "name": "anonichat-app"
  },
  "trace": {
    "id": "abc123"
  },
  "transaction": {
    "id": "def456"
  }
}
```

장점 
- 모든 서비스에서 표준화된 필드명을 가진다.
- kibana 대시보드와 호환성이 좋다.
- apm과 자동 연동이 된다.

일반 json 형식보다 확장성, 호환성이 좋은 ECS로 로그 형식을 변경했다.

## 설정 적용 및 테스트

### jenkins 빌드 및 컨테이너 재시작
spring boot에서 여러 설정을 변경했기 때문에 jenkins로 빌드 및 재실행을 해준다.

jenkins 빌드 중 문제 발생(docker-compose network)
[[jenkins docker-compose 네트워크 설정 문제]]


docker-compose 재시작
```
docker-compose down

docker-compose up
```

### 테스트
![[Pasted image 20250615161001.png]]
kibana에서 apm이 활성화 되었다.

이제 모니터링이 가능하도록 환경세팅 끝!
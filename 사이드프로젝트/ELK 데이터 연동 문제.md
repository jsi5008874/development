docker-compose로 logstash, elasticsearch, kibana를 모두 연동해서 이상없이 실행을 시켰는데
kibana ui에 데이터를 추가하라는 문구가 생겼다.

spring boot 어플리케이션에 접속을 해서 controller에 작성해놓은 로그가 쌓였을텐데 로그 검색은 안되고
설정하라는 화면만 보이고 있다.
![[Pasted image 20250608220538.png]]

중간에 문제가 있어서 elasticsearch에 로그가 쌓이지 않는것 같아 문제를 확인해 봤다.

### elasticsearch 인덱스 문제

elasticsearch 인덱스를 확인해봤다.
```
curl -X GET "localhost:9200/_cat/indices?v" health status index
```

![[Pasted image 20250608224617.png]]
**로그 데이터용 인덱스**(`main_log`)가 생성되지 않았다.
기본 시스템 로그만 생성되어있다.
우선 spring 앱과 logstash를 확인해봐야겠다.
### logback-spring.xml 설정 문제
logstash 에러로그
```
logstash_1       | [2025-06-08T11:49:33,676][WARN ][logstash.codecs.jsonlines][main][80f77f0ecd9b02bad7b731b91ce102e91b270d76b8623ef76f800e43725b74a4] JSON parse error, original data now in message field {:error=>#<LogStash::Json::ParserError: Unrecognized token 'Connection': was expecting ('true', 'false' or 'null') "; line: 1, column: 11]>, :data=>"Connection: keep-alive\r"}ive
```
해석해보면 spring boot 앱에서 logstash로 로그를 보낼 때 JSON으로 보내야하는데 HTTP 형식으로 와서 파싱에러가 발생했다.

그래서 logback-spring.xml을 확인해봤다.

```xml
<appender name="MAIN_LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">  
    <destination>logstash:5000</destination>  
    <encoder class="net.logstash.logback.encoder.LogstashEncoder" />  
    <keepAliveDuration>5 minutes</keepAliveDuration>  
</appender>

<logger name="MainServiceLogger" level="debug" additivity="false">  
    <appender-ref ref="AUCTION_LOGSTASH" />  <!-- 이름이 다름 -->  
    <appender-ref ref="CONSOLE" />  
</logger>
```
확인결과 json으로 인코딩해주는 것은 잘 들어가 있었지만
appender에서 설정한 name과 logger에서 설정한 name이 다르다.

appender에서는 MAIN_LOGSTASH인데 looger에서는 AUCTION_LOGSTASH로 되어있어 문제가 있었다.
**그래서 logger에 MAIN_LOGSTASH로 변경**

이제 jenkins로 변경된 코드로 다시 빌드하고 키바나에 접속해보자

### elasticsearch 라이센스 및 보안 문제
![[Pasted image 20250608221228.png]]
키바나에 접속 했는데 UI가 안보이고 오류로그가 JSON 형태로만 보였다.
찾아보니 elasticsearch 7.11+ 버전에서 발생하는 문제라고 한다.

docker-compose.yml을 수정할 필요가 있다.
우선 라이센스를 basic으로 명시해줘야하고 보안 설정을 false로 해야한다.

보안 설정을 해제하는 이유는 보안 설정을 하면 HTTPS 사용이 강제되어서 HTTP로는 접근이 불가능하기 때문이다.
**현재는 환경 구축 단계이고 HTTPS를 적용하지 않은 상태이기 때문에 나중에 HTTPS 설정을 하고 보안 설정을 다시 할 예정이다.**

docker-compose.yml 변경 사항
```yml
elasticsearch:
  image: docker.elastic.co/elasticsearch/elasticsearch:7.11.1
  environment:
    - discovery.type=single-node
    - xpack.security.enabled=false     # 보안 기능 비활성화
    - xpack.license.self_generated.type=basic  # 기본 라이센스 사용
  ports:
    - "9200:9200"
  networks:
    - elk
  volumes:
    - esdata:/usr/share/elasticsearch/data

kibana:
  image: docker.elastic.co/kibana/kibana:7.11.1
  ports:
    - "5601:5601"
  networks:
    - elk
  environment:
    - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    - server.host=0.0.0.0
    - xpack.security.enabled=false     # Kibana 보안 비활성화
    - xpack.license.self_generated.type=basic
```


```
# 현재 컨테이너들 정지 및 제거
docker-compose down

# 새로운 설정으로 시작
docker-compose up -d
```


이제 main_log가 생성되었다.
**spring > logstash > elasticsearch의 흐름이 정상 작동한다는 뜻이다.**
![[Pasted image 20250608230625.png]]


![[Pasted image 20250608230917.png]]
이제 정상적으로 검색이 된다!


## 결론
ELK의 흐름을 잘 이해하는게 관건이었던 것 같다.

spring boot > logstash > elasticsearch > kibana
 로그 생성          변환             로그 저장/ 조회    출력
 
네 가지의 요소가 각자의 역할을 하고 도커 컴포즈 내에서 내부 통신을 잘 하는지 확인하는것이 아주 중요했다.

결국 spring에서 부터 잘못 설정되었고 여기서 잘못되니 뒷 단의 요소들도 모두 오작동을 일으킨 것이다.

**처음부터 차근차근 생각해보고 풀어나가는 습관을 들이고 앞으론 실수가 없도록 설정파일들을 꼼꼼히 확인해봐야겠다.**


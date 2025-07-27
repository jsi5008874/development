
jenkins 빌드 중 mysql과 통신이 되지 않아 오류가 발생했다.

```
AnoniChatApplicationTests > contextLoads() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:180
        Caused by: org.springframework.beans.factory.BeanCreationException at AbstractAutowireCapableBeanFactory.java:1806
            Caused by: jakarta.persistence.PersistenceException at AbstractEntityManagerFactoryBean.java:421
                Caused by: org.hibernate.exception.JDBCConnectionException at SQLStateConversionDelegate.java:100
                    Caused by: com.mysql.cj.jdbc.exceptions.CommunicationsException at SQLError.java:174
                        Caused by: com.mysql.cj.exceptions.CJCommunicationsException at Constructor.java:500
                            Caused by: java.net.UnknownHostException at InetAddress.java:801
```

아무래도 elk-stack과 jenkins를 분리해서 사용하다보니 network 속성에 문제가 발생한 것 같다.

### jenkins docker-compose.yml 수정
기존 docker-compose
```yml
# jenkins docker-compose.yml
version: '3'

services:
  jenkins:
    image: ghcr.io/anonichat/app/jenkins-dood:v0.07
    container_name: jenkins-dood
    ports:
      - "8080:8080"

    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - jenkins_home:/home/hello
      - /home/hello/Desktop/AnoniChat/elk-stack:/home/hello/Desktop/AnoniChat/elk-stack
    networks:
      - data

networks:
  data:
    driver: bridge

volumes:
  jenkins_home:
    external: true

#ELK stack docker-compose.yml

 mysql:
    image: mysql:8.0
    container_name: mysql
    restart: unless-stopped
    ports:
      - "3306:3306"
    environment:
      - MYSQL_DATABASE=anonichat
      - MYSQL_ROOT_PASSWORD=비번
      - MYSQL_USER=anonichat
      - MYSQL_PASSWORD=비번
    volumes:
      - mysql_data:/home/hello/Desktop/AnoniChat/elk-stack/data/mysql
    networks:
      - elk
      - data

networks:
  elk:
  data:

  

volumes:
  esdata:
  mysql_data:
```
문제는 서로 다른 docker-compose라서 data라는 네트워크 이름만으로는 jenkins에서 인식을 못하는 것 같다.

도커 네트워크의 정확한 이름 확인
```
docker network ls

# 명령어 결과
NETWORK ID     NAME                DRIVER    SCOPE

b4db2da8fce5   bridge              bridge    local

64be00c696e5   elk-stack_data      bridge    local

13fe1529d8ed   elk-stack_default   bridge    local

8aaadfa031a2   elk-stack_elk       bridge    local

f30331b803ca   host                host      local

a07bf14ff474   none                null      local
```
elk-stack_data라는 이름의 네트워크를 사용해야한다.

```yml
version: '3'

services:
  jenkins:
    image: ghcr.io/anonichat/app/jenkins-dood:v0.07
    container_name: jenkins-dood
    ports:
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - jenkins_home:/home/hello
      - /home/hello/Desktop/AnoniChat/elk-stack:/home/hello/Desktop/AnoniChat/elk-stack

    networks:
      - data
      - elk-stack-data # 외부 네트워크 추가

networks:
  data:
    driver: bridge
  elk-stack-data:     
    external: true   # 외부 네트워크 사용
    name: elk-stack_data


volumes:
  jenkins_home:
    external: true
```

이렇게 external 속성을 추가하고 elk-stack_data 네트워크를 추가했다.

이후 재빌드

```
Starting a Gradle Daemon, 1 busy Daemon could not be reused, use --status for details
> Task :clean
> Task :compileJava
> Task :processResources
> Task :classes
> Task :resolveMainClassName
> Task :bootJar
> Task :jar
> Task :assemble
> Task :check
> Task :build

BUILD SUCCESSFUL in 17s
6 actionable tasks: 6 executed
```
성공적으로 빌드가 됐다.

### 문제점 분석
결국 서로 다른 docker 설정 파일에서 network 이름만 동일하게 지정한다고 해서 같은 네트워크를 사용하는 것이 아니었다.

**네트워크의 이름을 확인하고 외부 네트워크 속성을 정확히 지정해줘야 서로 다른 docker-compose로 생성된**
**컨테이너끼리 네트워크 통신이 가능하다.**
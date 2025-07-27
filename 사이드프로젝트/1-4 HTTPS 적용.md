
Nginx + certbot으로 https를 적용하는 과정이다.

### Nginx의 역할
HTTPS 적용의 입장에서만 보면 nginx는 특정 도메인으로 들어오는 모든 요청을 HTTPS로 리다이렉트 해주는 역할을 해주고
리버스 프록시로서 HTTPS를 HTTP로 변경해서 서버로 전달해준다(443 > 80)
또한 ssl 인증서를 관리하는 역할을 수행한다.

### certbot의 역할
- **인증서 발급** - Let's Encrypt CA에서 무료 SSL 인증서 받아옴
- **인증서 갱신** - 만료 전 자동으로 새 인증서 발급
- **파일 저장** - 인증서를 특정 디렉토리에 저장
- **nginx 알림** - 갱신 후 nginx에게 새 인증서 사용하라고 신호

certbot은 인증서를 발급/갱신하고 특정 디렉토리에 저장
그리고 nginx에게 알림을 주는 역할을 한다.
## 적용 순서
### 1. docker-compose.yml 수정

기존 도커 컴포즈 파일
```yml
version: '3'

  

services:

  elasticsearch:

    image: docker.elastic.co/elasticsearch/elasticsearch:7.11.1

    environment:

      - discovery.type=single-node

      - xpack.security.enabled=false

      - xpack.license.self_generated.type=basic

    ports:

      - "9200:9200"

    networks:

      - elk

    volumes:

      - esdata:/usr/share/elasticsearch/data

  

  logstash:

    image: docker.elastic.co/logstash/logstash:7.12.0

    ports:

      - "5044:5044"

      - "5000:5000"

    volumes:

      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf

    networks:

      - elk

    depends_on:

      - elasticsearch

  

  kibana:

    image: docker.elastic.co/kibana/kibana:7.11.1

    ports:

      - "5601:5601"

    networks:

      - elk

    environment:

      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200

      - server.host=0.0.0.0

      - xpack.security.enabled=false

      - xpack.license.self_generated.type=basic

    depends_on:

      - elasticsearch

  

  spring:

    image: ghcr.io/anonichat/app/anonichat

    ports:

      - "80:8080"

    environment:

      - ELASTICSEARCH_HOST=elasticsearch:9200

      - SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/anonichat?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=Asia/Seoul

      - SPRING_DATASOURCE_USERNAME=anonichat

      - SPRING_DATASOURCE_PASSWORD=비번

    depends_on:

      - elasticsearch

      - mysql

    networks:

      - elk

      - data

  

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

기존 도커 컴포즈 파일에 nginx와 certbot을 추가한다.

**수정한 도커 컴포즈 파일**
```yml
version: '3'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.11.1
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - xpack.license.self_generated.type=basic
    ports:
      - "9200:9200"
    networks:
      - elk
    volumes:
      - esdata:/usr/share/elasticsearch/data

  logstash:
    image: docker.elastic.co/logstash/logstash:7.12.0
    ports:
      - "5044:5044"
      - "5000:5000"
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    networks:
      - elk
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:7.11.1
    ports:
      - "5601:5601"
    networks:
      - elk
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
      - server.host=0.0.0.0
      - xpack.security.enabled=false
      - xpack.license.self_generated.type=basic
    depends_on:
      - elasticsearch

  # Spring Boot 앱 (포트 변경 중요!)
  spring:
    image: ghcr.io/anonichat/app/anonichat
    # 외부 포트 제거 - nginx를 통해서만 접근
    expose:
      - "8080"
    environment:
      - ELASTICSEARCH_HOST=elasticsearch:9200
      - SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/anonichat?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=Asia/Seoul
      - SPRING_DATASOURCE_USERNAME=anonichat
      - SPRING_DATASOURCE_PASSWORD=비번
    depends_on:
      - elasticsearch
      - mysql
    networks:
      - elk
      - data

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

  # Nginx 리버스 프록시
  nginx:
    image: nginx:alpine
    container_name: nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
      - ./certbot/conf:/etc/letsencrypt
      - ./certbot/www:/var/www/certbot
    depends_on:
      - spring
    networks:
      - elk  # Spring과 같은 네트워크에 연결
    command: ["nginx", "-g", "daemon off;"]

  # Certbot SSL 인증서 관리
  certbot:
    image: certbot/certbot
    container_name: certbot
    restart: unless-stopped
    volumes:
      - ./certbot/conf:/etc/letsencrypt
      - ./certbot/www:/var/www/certbot
      - /var/run/docker.sock:/var/run/docker.sock
    entrypoint: "/bin/sh"
    command: > # ssl 인증서 갱신시에만 nginx 설정 reload하는 명령어
      -c "
      apk add --no-cache docker-cli;
      while :; do
        echo 'Checking for certificate renewal...';
        certbot renew --deploy-hook 'docker exec nginx nginx -s reload';
        sleep 12h;
      done
      "

networks:
  elk:
  data:

volumes:
  esdata:
  mysql_data:
```

**spring의 port 변경**
```yml
# 기존 설정
ports: - "80:8080" # 호스트포트:컨테이너포트

# 변경된 설정
expose: - "8080" # 컨테이너포트만

# nginx의 설정
ports:
      - "80:80"
      - "443:443"
```
기존 설정은 외부를 통해서 컨테이너에 접근이 가능했다.
브라우저 → 서버:80 → Spring Boot:8080
이렇게 외부에서 80 포트로 요청이 들어오면 호스트(인스턴스)의 80포트를 통해 스프링 컨테이너의 8080포트로 전달해주는 방식

변경된 설정은 컨테이너의 8080만 열어놓는다.
이렇게 되면 호스트의 80포트로 들어오는 요청은 무조건 nginx를 거쳐서 들어오게 된다.
브라우저 → 서버:80 → nginx:80 → spring:8080

HTTP 요청이 프록시를 거쳐서 오게끔 구성한 설정이다.
### 2. 디렉토리 생성
```
# docker-compose.yml이 있는 폴더에서 실행
mkdir nginx
# nginx 하위 폴더 생성
mkdir conf.d

# docker-compose.yml이 있는 폴더에서 실행
mkdir certbot
#certbot 하위 폴더 생성
mkdir conf
mkdir www
```


### 3. Nginx 설정 파일 생성 및 ssl 인증서 발급

**인증서 발급 전 임시 설정**
```
upstream spring-backend {
    server spring:8080;
}

# HTTP 서버 (인증서 발급 전 임시 설정)
server {
    listen 80;
    server_name anonichat.world www.anonichat.world;
    server_tokens off;

    # Let's Encrypt 도메인 인증용 경로
    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    # AnoniChat 애플리케이션 (HTTP로 임시 서비스)
    location / {
        proxy_pass http://spring-backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Host $server_name;
        
        # WebSocket 지원 (채팅 기능용)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```


**도커 컴포즈 재시작**
```
docker-compose up -d
```

**ssl 인증서 발급**
```
docker run --rm -it \                                                                                                                         --name certbot \                                               

  -v $(pwd)/certbot/conf:/etc/letsencrypt \

  -v $(pwd)/certbot/www:/var/www/certbot \

  certbot/certbot \

  certonly --webroot \

  --webroot-path /var/www/certbot \

  --email jsi50069@gmail.com \

  --agree-tos \

  --no-eff-email \

  -d anonichat.world \

  -d www.anonichat.world
```
certbot으로 ssl 인증서를 받음

**인증서 확인**
```
docker-compose exec certbot ls -la /etc/letsencrypt/live/anonichat.world/
```

**certbot 재시작**
```
docker-compose up -d certbot
```

**nginx 설정 파일 변경**
```
upstream spring-backend {

    server spring:8080;

}

  

# HTTP → HTTPS 강제 리다이렉트

server {

    listen 80;

    server_name anonichat.world www.anonichat.world;

    server_tokens off;

  

    # Let's Encrypt 경로만 HTTP 허용

    location /.well-known/acme-challenge/ {

        root /var/www/certbot;

    }

  

    # 나머지 모든 요청은 HTTPS로 강제 리다이렉트

    location / {

        return 301 https://$host$request_uri;

    }

}

  

# HTTPS 서버

server {

    listen 443 ssl;

    http2 on;

    server_name anonichat.world www.anonichat.world;

    server_tokens off;

  

    ssl_certificate /etc/letsencrypt/live/anonichat.world/fullchain.pem;

    ssl_certificate_key /etc/letsencrypt/live/anonichat.world/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;

    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384;

    ssl_prefer_server_ciphers off;

    ssl_session_cache shared:SSL:10m;

    ssl_session_timeout 10m;

    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    add_header X-Frame-Options DENY always;

    add_header X-Content-Type-Options nosniff always;

  

    location / {

        proxy_pass http://spring-backend;

        proxy_set_header Host $host;

        proxy_set_header X-Real-IP $remote_addr;

        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_set_header X-Forwarded-Host $server_name;

        proxy_http_version 1.1;

        proxy_set_header Upgrade $http_upgrade;

        proxy_set_header Connection "upgrade";

    }

}
```


### 4. 재배포 및 테스트

**nginx 설정 파일 테스트**
```
docker-compose exec nginx nginx -t

# 명령어 결과
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

**nginx 재시작**
```
docker-compose exec nginx nginx -s reload
```

**테스트**
```
curl -I http://anonichat.world
# 명령어 결과
HTTP/1.1 301 Moved Permanently

**Server**: nginx

**Date**: Thu, 12 Jun 2025 16:24:54 GMT

**Content-Type**: text/html

**Content-Length**: 162

**Connection**: keep-alive

**Location**: https://anonichat.world/
```
Location을 보면 https로 전환되는것을 볼 수 있다.

![[Pasted image 20250613013521.png]]
브라우저에서도 HTTPS로 변경된 것을 확인
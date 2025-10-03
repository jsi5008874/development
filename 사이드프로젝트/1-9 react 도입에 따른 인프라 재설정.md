기존 인프라 세팅은 spring boot  기준으로 설정되어 있었다.
프론트엔드 도입에 따라 인프라 세팅을 변경해야한다.
1. jenkins item 추가(프론트엔드 프로젝트)
2. docker-compose 추가
3. nginx 설정 변경

## jsenkins item 추가 및 ci/cd 파이프라인 작성

### jenkins item 추가
anoni-react-app item 생성
![[Pasted image 20250728105218.png]]

### node.js 플러그인 설치 및 글로벌 툴 설정
**플러그인 설치**
![[Pasted image 20250728105332.png]]
jenkins 관리 > 플러그인 > 플러그인 설치

**글로벌 툴 설정**
![[Pasted image 20250728105443.png]]
jenkins 관리 > tools > NodeJS installations

![[Pasted image 20250728105524.png]]
Name 지정(젠킨스 파이프라인 스크립트에서 사용할 name) > node version 선택 > 저장

이렇게 툴에 지정을 해 놓으면 젠킨스 서버에 node.js를 따로 설치하지 않아도 빌드가 가능하다.

### jenkins 파이프라인 스크립트 작성
```
pipeline {
    agent any
    
    tools {
        nodejs 'react'  // Global Tool Configuration에서 설정한 이름
    }

    environment {
        DOCKER_HUB_REPO = 'ghcr.io/anonichat/web/react-app' // 이미지 레지스트리
        DOCKER_IMAGE_TAG = '${BUILD_NUMBER}'
        DOCKER_LATEST_TAG = 'latest'
        DOCKER_HUB_CREDENTIALS = 'gitPackage'
        CONTAINER_NAME = 'anonichat-frontend'
        CONTAINER_PORT = '3000:3000'
        NODE_VERSION = '22'
    }

    stages {
        stage('1. Git Clone') {
            steps {
                echo '📂 GitHub에서 프론트엔드 최신 코드 가져오기...'
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/prod']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/AnoniChat/Web.git',  // 프론트엔드 저장소
                        credentialsId: 'githubRepository'
                    ]]
                ])
                sh 'git log --oneline -5'
            }
        }

        stage('2. Build') {
            steps {
                echo '🔨 React 애플리케이션 빌드 중...'
                script {
                    // frontend 폴더가 있는지 확인 (프로젝트 구조에 따라)
                    def frontendDir = fileExists('frontend') ? 'frontend' : '.'
                    
                    dir(frontendDir) {
                        if (fileExists('package.json')) {
                            sh """
                            echo "Node.js 버전 확인:"
                            node --version
                            npm --version
                    
                            npm run lint
                        } else {
                            error('package.json 파일이 없습니다. React 프로젝트인지 확인하세요!')
                        }
                    }
                }
            }
        }

        stage('3. Docker Image Build') {
            steps {
                echo '🐳 Docker 이미지 빌드 중...'
                script {
                    def frontendDir = fileExists('frontend') ? 'frontend' : '.'
                    
                    dir(frontendDir) {
                        // 빌드 결과 확인 (여러 가능한 위치 체크)
                        def buildExists = sh(
                            script: '''
                            if [ -d ".next/standalone" ]; then
                                echo "standalone build found"
                                exit 0
                            elif [ -d ".next" ]; then
                                echo "regular next build found"
                                exit 0
                            elif [ -d "build" ]; then
                                echo "react build found"
                                exit 0
                            elif [ -d "dist" ]; then
                                echo "dist build found"  
                                exit 0
                            else
                                echo "no build found"
                                exit 1
                            fi
                            ''', 
                            returnStatus: true
                        )
                        
                        if (buildExists == 0) {
                            echo "빌드 결과 확인 완료"
                            // .next 디렉토리 내용 확인
                            sh "ls -la .next/ || echo 'no .next directory'"
                            
                            sh """
                            docker build -t ${DOCKER_HUB_REPO}:${DOCKER_IMAGE_TAG} .
                            docker tag ${DOCKER_HUB_REPO}:${DOCKER_IMAGE_TAG} ${DOCKER_HUB_REPO}:${DOCKER_LATEST_TAG}
                            """
                        } else {
                            error('프론트엔드 빌드 결과를 찾을 수 없습니다!')
                        }
                    }
                }
            }
        }

        stage('4. Docker Image Push') {
            steps {
                echo '📤 GHCR에 이미지 푸시 중...'
                script {
                    withCredentials([usernamePassword(credentialsId: "githubPackage", 
                                                    passwordVariable: 'DOCKER_PASSWORD', 
                                                    usernameVariable: 'DOCKER_USERNAME')]) {
                        sh """
                        echo ${DOCKER_PASSWORD} | docker login ghcr.io -u ${DOCKER_USERNAME} --password-stdin
                        docker push ${DOCKER_HUB_REPO}:${DOCKER_IMAGE_TAG}
                        docker push ${DOCKER_HUB_REPO}:${DOCKER_LATEST_TAG}
                        docker logout
                        """
                    }
                }
                echo "✅ 이미지 푸시 완료!"
            }
        }

        stage('5. Deploy') {
            steps {
                echo '🚀 최신 이미지로 배포 중...'
                script {
                    try {
                        sh "docker stop ${CONTAINER_NAME} || true"
                        sh "docker rm ${CONTAINER_NAME} || true"
                        sh "docker rmi ${DOCKER_HUB_REPO}:${DOCKER_LATEST_TAG} || true"
                        sh "docker pull ${DOCKER_HUB_REPO}:${DOCKER_LATEST_TAG}"
                        sh """
                        docker run -d \\
                          --name ${CONTAINER_NAME} \\
                          --restart always \\
                          --network elk-stack_elk \\
                          -e NODE_ENV=production \\
                          -e NEXT_PUBLIC_API_URL=https://anonichat.world/api \\
                          --expose 3000 \\
                          ${DOCKER_HUB_REPO}:${DOCKER_LATEST_TAG}
                        """
                        sh "sleep 10"
                        sh "docker ps | grep ${CONTAINER_NAME}"
                    } catch (Exception e) {
                        echo "❌ 배포 실패: ${e.getMessage()}"
                        throw e
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                node {  // node context 추가
                    echo '🧹 빌드 후 정리 작업...'
                    sh 'docker image prune -f || true'
                }
            }
        }
        success {
            echo '🎉 프론트엔드 CI/CD 파이프라인이 성공적으로 완료되었습니다!'
            echo '✅ Frontend: https://anonichat.world'
        }
        failure {
            echo '❌ 프론트엔드 CI/CD 파이프라인이 실패했습니다.'
        }
    }
}
```
빌드 및 배포 자체는 이상 없이 되었지만 빌드가 너무 오래걸린다는 단점이 있었다.
매번 빌드할 때마다 npm ci로 모든 의존성을 설치하다보니 시간이 오래걸리는 현상이 발생했다.(최종 빌드 및 배포 6분)

따라서 npm 캐시를 활용하여 빌드를 최적화했다.

### npm 빌드 최적화
**1. 도커 파일 수정**
```
COPY package.json yarn.lock* package-lock.json* pnpm-lock.yaml* ./
```
> 의존성 파일 복사

```
RUN \

  if [ -f yarn.lock ]; then \

    yarn install --frozen-lockfile --network-timeout 300000; \

  elif [ -f package-lock.json ]; then \

    npm ci --ignore-scripts --no-audit --cache /tmp/.npm-cache; \

  elif [ -f pnpm-lock.yaml ]; then \

    yarn global add pnpm && pnpm i --frozen-lockfile; \

  else \

    echo "Lockfile not found." && exit 1; \

  fi
```
> 의존성 변경시에만 의존성 새로 설치

```
COPY --from=deps /app/node_modules ./node_modules
```
> 의존성 변경점이 없으면 캐시된 의존성을 그대로 사용

도커 빌드 시 의존성 변경점을 확인하고 변경이 없으면 캐시에 저장된 의존성을 그대로 사용하고
변경 시에는  npm ci로 packge-lock.json의 의존성을 다시 설치

**2.젠킨스 docker-compose.yml 수정**
```
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - jenkins_home:/home/hello
      - /home/hello/Desktop/AnoniChat/elk-stack:/home/hello/Desktop/AnoniChat/elk-stack
      - ./jenkins-npm-cache:/home/jenkins/.npm
```
volumes에 npm cache 관련 내용 추가(마운트 볼륨 추가)

**3. 젠킨스 스크립트 수정**
```
        stage('2. 빠른 검증') {
            steps {
                echo '🔍 코드 품질 검사 및 캐시 준비...'
                script {
                    def frontendDir = fileExists('frontend') ? 'frontend' : '.'
                    
                    dir(frontendDir) {
                        if (fileExists('package.json')) {
                            sh """
                            echo "Node.js 버전 확인:"
                            node --version
                            npm --version
                            
                            echo "npm 캐시 설정 (볼륨 마운트 활용)..."
                            npm config set cache ~/.npm
                            npm config set prefer-offline true
                            npm config set audit false
                            npm config set fund false
                            
                            echo "캐시 설정 완료: \$(npm config get cache)"
                            echo "캐시 디렉토리 상태: \$(ls -la ~/.npm 2>/dev/null | wc -l) 개 파일"
                            
                            echo "의존성 설치 중 (호스트 볼륨 캐시 활용)..."
                            npm ci --prefer-offline
                            
                            echo "코드 품질 검사 실행..."
                            if npm run | grep -q "lint"; then
                                npm run lint || echo "⚠️ Lint 경고가 있지만 계속 진행합니다."
                            else
                                echo "lint 스크립트가 없습니다. 건너뜁니다."
                            fi
                            """
                        } else {
                            error('package.json 파일이 없습니다. React 프로젝트인지 확인하세요!')
                        }
                    }
                }
            }
        }

```
새로 설정한 jenkins 볼륨마운트를 활용하여 npm cache로 더 빠르게 빌드 수행

### 결과
![[Pasted image 20250728143641.png]]
6분에서 1분으로 빌드 시간 감소
## docker-compose.yml 수정
### docker-compose.yml 수정
```yml
  # React 프론트엔드 서비스 추가
  react:
    image: ghcr.io/anonichat/app/frontend:latest
    container_name: anonichat-frontend
    expose:
      - "3000"
    environment:
      - NODE_ENV=production
      - NEXT_PUBLIC_API_URL=https://anonichat.world/api
    depends_on:
      - spring
    networks:
      - elk
    restart: unless-stopped
```
docker-compose.yml에 react 추가

## nginx 설정 수정(conf.d 파일)
### upstream 추가
```

# Frontend upstream (Jenkins에서 생성하는 컨테이너 이름 반영)
upstream react-frontend {
    server anonichat-frontend:3000;  # Jenkins에서 생성하는 컨테이너 이름
}
```
frontend upstream 추가

### http_origin 및 location 설정(백엔드 api)
```


# HTTPS 서버
server {
    ........
    
    # API 요청 - Spring Boot로 라우팅
    location /api/ {
    # 허용된 Origin 검증을 위한 변수 설정
     set $cors_origin "";
      # 허용된 도메인만 CORS 허용
       if ($http_origin = "https://anonichat.world") { set $cors_origin "https://anonichat.world"; }
       if ($http_origin = "https://www.anonichat.world") { set $cors_origin "https://www.anonichat.world"; }
        proxy_pass http://spring-backend/api/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Host $server_name;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
```
/api/가 포함된 url은 백엔드 서버로 넘기는 설정(RequestMapping과 비슷한 역할)

### CORS 헤더 설정
```
        # CORS 헤더 (필요한 경우)
        proxy_hide_header Access-Control-Allow-Origin; #CORS 헤더 중복 방지용
        add_header Access-Control-Allow-Origin $http_origin always; # 동적 출처 허용
        add_header Access-Control-Allow-Credentials true always; # 쿠키, 인증 헤더를 포함한 요청 허용
        # 허용할 HTTP 메서드 명시
        add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS, PATCH" always;
        # 클라이언트가 보낼 수 있는 헤더 명시
        add_header Access-Control-Allow-Headers "Accept, Authorization, Cache-Control, Content-Type, DNT, If-Modified-Since, Keep-Alive, Origin, User-Agent, X-Requested-With" always;
```
 **CORS (Cross-Origin Resource Sharing)**
**"다른 출처의 리소스를 공유할 수 있게 해주는 메커니즘"**

CORS가 필요한 이유?
 **Same-Origin Policy (동일 출처 정책) 때문이다.**
> 브라우저의 기본 보안 정책으로, 다른 출처로의 요청을 차단

Same-Origin Policy가 없다면 악성 사이트에서 다른 사이트 API를 마음대로 호출 가능

**예시**
```
// 현재 페이지: https://anonichat.world

// ✅ 같은 출처 - 허용
fetch('/api/users')                           // 같은 도메인
fetch('https://anonichat.world/api/users')    // 완전히 동일

// ❌ 다른 출처 - 차단
fetch('http://anonichat.world/api/users')     // 프로토콜 다름 (https → http)
fetch('https://api.anonichat.world/users')    // 서브도메인 다름
fetch('https://anonichat.world:8080/users')   // 포트 다름
fetch('https://google.com/api')               // 완전히 다른 도메인
```
이렇게 출처가 다르면 브라우저에서 차단을 하기 때문에 우회하도록 해주는게 CORS 헤더이다.

서버에서 허용 헤더를 보내주면 브라우저가 요청을 허용한다.
```
HTTP/1.1 200 OK Access-Control-Allow-Origin: https://anonichat.world
Access-Control-Allow-Methods: GET, POST
Content-Type: application/json
{"message": "CORS 덕분에 요청 성공!"}
```

현재 아키텍처에서는 CORS가 딱히 필요 없긴하다.
모든 요청이 anonichat.world를 통해서 도달하기 때문에 필요없지만 추후 확장성을 고려하여 추가했다.
(외부 api 통신 같이 다른 url로 요청이 들어올 때 필요)
### Prefligt 설정
```
        # Preflight 요청 처리
        if ($request_method = 'OPTIONS') {
            add_header Access-Control-Allow-Origin $http_origin;
            add_header Access-Control-Allow-Credentials true;
            add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS, PATCH";
            add_header Access-Control-Allow-Headers "Accept, Authorization, Cache-Control, Content-Type, DNT, If-Modified-Since, Keep-Alive, Origin, User-Agent, X-Requested-With";
            add_header Access-Control-Max-Age 1728000;
            add_header Content-Type 'text/plain charset=UTF-8';
            add_header Content-Length 0;
            return 204;
        }
    }
```
**Preflight**
브라우저가 실제 요청을 보내기 전에 "이 요청을 보내도 괜찮은지" 확인하는 사전 요청

**Preflight는 CORS의 보안 장치**
CORS가 다른 도메인의 요청을 허용하는 역할인데 preflight는 위험한 요청은 미리 확인하고 차단하는 용도

**예시**
```
#Preflight 없을 때
사용자가 evil.com 방문 → 즉시 DELETE 요청 전송 → 데이터 삭제됨

#Preflight 있을 때
사용자가 evil.com 방문 → OPTIONS 요청 먼저 → 서버가 "evil.com은 안돼!" → 차단
```

즉 CORS만 허용하면 악성 요청에 대비하기 어렵지만 Preflight가 미리 악성요청을 필터링해주는 개념


### 정적파일 및 리액트 라우팅 설정
```
    # Next.js 정적 파일 최적화
    location /_next/static/ {
        proxy_pass http://react-frontend;
        proxy_set_header Host $host;
        expires 1y;
        add_header Cache-Control "public, immutable";
        add_header X-Cache-Status "STATIC";
    }

    # Next.js 이미지 최적화
    location /_next/image {
        proxy_pass http://react-frontend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # 파비콘 및 기타 정적 파일
    location ~* \.(ico|css|js|gif|jpe?g|png|svg|woff|woff2|ttf|eot)$ {
        proxy_pass http://react-frontend;
        proxy_set_header Host $host;
        expires 1y;
        add_header Cache-Control "public, immutable";
        add_header X-Cache-Status "STATIC";
    }

    # 나머지 모든 요청 - React 프론트엔드로 라우팅
    location / {
        proxy_pass http://react-frontend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Host $server_name;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # SPA 라우팅을 위한 fallback 처리
        proxy_intercept_errors on;
        error_page 404 = @fallback;
    }
```
정적 파일 및 react 프로젝트로 라우팅 설정

### fallback 설정
```
    # SPA fallback 처리
    location @fallback {
        proxy_pass http://react-frontend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
**SPA fallback**
Single Page Application에서 클라이언트 라우팅을 지원하기 위해, 존재하지 않는 경로에 대해 메인 HTML 파일(index.html)을 제공하는 메커니즘

**fallback 예시**
```
# fallback 없을 때
1. 사용자가 직접 URL 입력 https://anonichat.world/chat/123 
2. nginx: "/chat/123 파일이 없네? 404!" 
3. 사용자: "사이트가 망가졌나?"

# fallback 해결
1. nginx: "/chat/123 없음 → 404"
2. nginx: "어? 404니까 @fallback 실행"
3. @fallback: "React에게 다시 요청 (루트로)" 
4. React: "index.html 줄게" 
5. 브라우저: "React Router야, /chat/123 처리해줘" 
6. 사용자: "채팅방 나왔다!"
```

**설정 별 역할**
`location @fallback`
- `@` = Named Location (에러 처리 전용)
- 404 에러가 발생했을 때만 실행되는 특별한 location

`proxy_pass http://react-frontend`
- React 컨테이너에게 "다시 한 번 요청해줘"
- 하지만 이번에는 `/` (루트)로 요청

`proxy_set_header`들
- **Host**: "요청 도메인이 anonichat.world야"
- **X-Real-IP**: "실제 사용자 IP는 이거야"
- **X-Forwarded-For**: "프록시 경로는 이래"
- **X-Forwarded-Proto**: "원본은 HTTPS였어"


**결론**: SPA Fallback 덕분에 React 앱의 모든 페이지가 **북마크 가능하고 직접 접속 가능**
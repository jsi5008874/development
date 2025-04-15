지속적 통합(CI) 도구로 코드 변경사항을 자동으로 빌드, 테스트, 패키징하는 역할
개발자가 코드를 git에 푸시하면 jenkins가 이를 감지하고 빌드 프로세스를 시작

jenkins가 git(저장소)를 계속 감시하면서 변경을 감지하면 활동한다.

이를 구현하는 방법은 크게 두 가지로
1.  폴링(Polling) : 일정 시간 간격으로 저장소의 변경사항 확인
2.  웹훅(Webhook) : 저장소에 변경사항이 발생했을 때 jenkins에 즉시 알려주는 방식(주로 사용되는 방식)

**이를 설정하는 방법은 jenkins의 파이프 라인 설정을 통해 이루어진다.**
파이프라인 설정 : 소프트웨어 빌드, 테스트, 배포 등의 전체 과정을 코드로 정의 하는 것

**jenkins 파이프라인의 유형**
1.  선언적 파이프라인(Declarative Pipeline) : 구조화된 접근 방식으로 사용하기 쉬움(일반적으로 많이 사용)
2. 스크립트 파이프라인(Script Pipeline) : Groovy 스크립트 기반, 더 유연하지만 복잡한 구문

### 선언적 파이프라인 예시(feat. 클로드)
```Groovy
pipeline {
    agent any
    
    // 웹훅은 외부에서 트리거하므로 triggers 블록이 필요 없습니다.
    // Git 저장소에서 웹훅을 Jenkins 서버로 설정해야 합니다.
    
    environment {
        // Maven 프로젝트용 환경 변수 설정
        MAVEN_HOME = tool 'Maven 3.8.4'
        PATH = "${MAVEN_HOME}/bin:${env.PATH}"
        
        // Docker 이미지 관련 환경 변수
        DOCKER_REGISTRY = "registry.example.com"
        IMAGE_NAME = "my-spring-app"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                // 코드 체크아웃
                checkout scm
                
                // 어떤 브랜치/커밋이 빌드되는지 로그 출력
                sh 'git log -1'
                echo "Building branch: ${env.BRANCH_NAME}"
            }
        }
        
        stage('Build') {
            steps {
                // Maven으로 빌드
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Unit Tests') {
            steps {
                // 단위 테스트 실행
                sh 'mvn test'
            }
            post {
                always {
                    // JUnit 테스트 결과 수집
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Code Quality') {
            steps {
                // SonarQube 분석 실행
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        
        stage('Build Docker Image') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                // Docker 이미지 빌드
                sh "docker build -t ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ."
                sh "docker tag ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_REGISTRY}/${IMAGE_NAME}:latest"
            }
        }
        
        stage('Push Docker Image') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                // Docker 레지스트리 인증
                withCredentials([usernamePassword(credentialsId: 'docker-registry-credentials', 
                                              usernameVariable: 'DOCKER_USER', 
                                              passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh "echo ${DOCKER_PASSWORD} | docker login ${DOCKER_REGISTRY} -u ${DOCKER_USER} --password-stdin"
                    sh "docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:latest"
                }
            }
        }
        
        stage('Update Kubernetes Manifests') {
            when {
                branch 'main'
            }
            steps {
                // K8s 매니페스트 저장소 체크아웃
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/your-org/k8s-manifests.git',
                        credentialsId: 'git-credentials'
                    ]]
                ])
                
                // 이미지 태그 업데이트
                sh """
                    sed -i 's|image: ${DOCKER_REGISTRY}/${IMAGE_NAME}:.*|image: ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}|g' deployment.yaml
                    git config user.email "jenkins@example.com"
                    git config user.name "Jenkins"
                    git add deployment.yaml
                    git commit -m "Update image to ${IMAGE_TAG}"
                    git push origin main
                """
                
                // ArgoCD가 이 변경사항을 감지하여 배포를 진행합니다
            }
        }
    }
    
    post {
        success {
            echo "Pipeline completed successfully!"
            // Slack이나 이메일로 성공 알림 보내기
            slackSend(color: 'good', message: "Build succeeded: ${env.JOB_NAME} #${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        failure {
            echo "Pipeline failed!"
            // Slack이나 이메일로 실패 알림 보내기
            slackSend(color: 'danger', message: "Build failed: ${env.JOB_NAME} #${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        always {
            // 작업 공간 정리
            cleanWs()
        }
    }
}
```

웹훅 방식은 코드의 변경을 확인하는 설정을 파이프라인에서 설정하지 않고 jenkins에 github 플러그인을 설치하여 github 섹션에 웹훅 설정

폴링 방식은 파이프라인에서 설정
```Groovy
triggers { pollSCM('H/15 * * * *') // 15분마다 코드 변경 확인 }
```
파이프라인에 트리거를 설정해서 젠킨스가 일정 시간 간격으로 코드 저장소의 변경사항을 확인하도록 설정

이렇게 파이프라인 설정을 통해 빌드, 테스트, 컨테이너 이미지 생성, 도커 매니페스트 업데이트등 모든 것을 설정한다.

파이프라인의 코드를 보면
**빌드 > 테스트 > 도커/컨테이너 이미지 빌드 > 도커/컨테이너 레지스트리 푸시 > 쿠버네티스 매니페스트 업데이트**
순서로 진행된다.

각 단계에 대해 알아보자

## 도커/컨테이너 이미지 빌드

### 도커/컨테이너 이미지 란?
도커/컨테이너 이미지는 애플리케이션과 그 실행에 필요한 모든 것을 포함하는 가볍고 독립적인 패키지
코드, 런타임, 시스템 도구, 시스템 라이브러리, 설정 등 실행에 필요한 것들을 포함하고 있다.

코드 : 애플리케이션의 코드

런타임 : 애플리케이션이 실행하는데 필요한 소프트웨어 환경, Java의 JVM, Node.js의 Node.js 런타임 등

시스템 도구 : 운영 체제 기능을 제공하는 유틸리티 프로그램
			bash, ping, curl 같은 명령어 도구(운영, 디버깅, 모니터링 등에 사용된다.)

시스템 라이브러리 : 애플리케이션이 운영 체제의 기능을 사용할 수 있게 해주는 공유 라이브러리 파일들
				 libc(C 표준 라이브러리), libssl(암호화 기능) 등 파일 시스템 접근, 네트워크 통신 등의 기능 제공

설정 : 애플리케이션의 동작 방식을 정의하는 파라미터, 환경 변수, 구성 파일(properties, yml, json 등)
	 spring boot의 application.yml 같은 설정 파일

이렇게 구성된 도커/컨테이너 이미지는 애플리케이션을 실행하는데 필요한 모든 구성요소가 포함되어 있어
매우 독립적이며 이식성도 좋다.
또한 생성 후 변경 되지 않는 읽기 전용 템플릿으로 불변성을 가지고 있다.

### 도커 이미지, 컨테이너 이미지의 차이점
도커 이미지와 컨테이너 이미지는 사실상 같은 개념을 가리키는 용어이다.

**도커 이미지**
Doker라는 특정 기술/플랫폼을 통해 생성되고 관리되는 이미지를 뜻한다.
Doker에 종속적인 용어

**컨테이너 이미지**
보다 일반적인 용어로 컨테이너 기술을 사용하는 모든 플랫폼의 이미지를 뜻한다.
OCI(Open Container Initiative) 표준을 따르는 모든 이미지를 지칭

도커도 컨테이너 기술 중 하나이기 때문에 거의 똑같은 의미를 내포하고 있다.

### 도커/컨테이너 이미지 빌드의 과정

**docker file**
컨테이너 이미지에 들어갈 내용은 주로 docker file이라는 파일에 정의 된다.
jenkins는 이 doker file을 기반으로 이미지를 빌드한다.

doker file 예시
```json
# 베이스 이미지 지정 (여기에 기본 런타임, 일부 시스템 도구 및 라이브러리 포함)
FROM openjdk:11-jdk-slim

# 작업 디렉토리 설정
WORKDIR /app

# 시스템 도구 및 라이브러리 설치
RUN apt-get update && apt-get install -y \
    curl \
    netcat \
    && rm -rf /var/lib/apt/lists/ *

# 애플리케이션 JAR 파일 복사 (빌드 결과물)
COPY target/myapp.jar /app/

# 환경 변수 설정 (설정)
ENV SPRING_PROFILES_ACTIVE=production

# 포트 노출
EXPOSE 8080

# 실행 명령어
CMD ["java", "-jar", "myapp.jar"]
```

이런 doker file은 프로젝트 소스 코드 저장소에 함께 보관되며 jenkins가 소스 코드와 함께 가져온다.

doker file에 도커/컨테이너 이미지의 내용을 정의하고 jenkins가 이 명세를 따라 이미지를 자동으로 빌드

이미지 빌드의 과정을 정리하자면
1. 젠킨스가 소스 코드 저장소에서 소스 코드와 함께 doker file을 가져온다.
2.  jenkins가 파이프라인 설정에 따라 이미지를 빌드하는데 가져온 doker file의 명세를 이용하여 자동으로 빌드


## 도커/컨테이너 레지스트리 푸시

### 도커/컨테이너 레지스트리
도커/컨테이너 이미지를 저장하고 관리하는 중앙 저장소
git 코드를 위한 저장소와 유사한 개념

**도커 컨테이너 레지스트리의 역할**
- **이미지 저장 및 관리**: 빌드된 컨테이너 이미지를 중앙 위치에 저장
- **버전 관리**: 이미지의 여러 버전을 태그(tag)를 통해 관리
- **접근 제어**: 이미지에 대한 접근 권한을 관리
- **이미지 배포**: CI/CD 파이프라인과 쿠버네티스 같은 오케스트레이션 시스템이 레지스트리에서 이미지를 가져와 실행
- **취약점 스캔**: 많은 레지스트리가 저장된 이미지의 보안 취약점을 자동으로 스캔하는 기능을 제공


Doker hub, AWS ECR(Elastic Container Registry), Harbor 등이 있다.

젠킨스가 이미지를 빌드/생성하면 도커/컨테이너 레지스트리에 푸시 > 쿠버네티스 같은 배포 플랫폼이 레지스트리에서
이미지를 가져와서 실행

## 쿠버네티스 매니페스트 업데이트

### 쿠버네티스 매니페스트(manifest)란?
쿠버네티스 클러스터에 생성하려는 리소스와 그 상태를 선언적으로 정의하는 YAML, Json 형식의 파일
쿠버네티스가 어떻게 애플리케이션을 배포하고 관리하는지에 대한 내용을 담고 있다.

### 쿠버네티스 매니페스트의 구성
1.  리소스 종류(kind) : Deployment, Service, Pod 등 생성할 쿠버네티스 리소스의 유형
2.  메타데이터 : 리소스의 이름, 네임스페이스, 레이블 등 식별 정보
3.  스펙 : 리소스의 원하는 상태에 대한 자세한 정의(컨테이너 이미지, 포트 설정, 리소스 제한 > CPU, 메모리 제한 등)

세 가지 외에도 여러 구성요소가 존재한다.

**매니페스트 예시**
```text
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-boot-app
  namespace: my-application
spec:
  replicas: 3
  selector:
    matchLabels:
      app: spring-boot-app
  template:
    metadata:
      labels:
        app: spring-boot-app
    spec:
      containers:
      - name: spring-boot-app
        image: myregistry/spring-boot-app:1.0.0
        ports:
        - containerPort: 8080
        resources:
          limits:
            cpu: "500m"
            memory: "512Mi"
          requests:
            cpu: "200m"
            memory: "256Mi"
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "production"
```

매니페스트 파일은 Git 저장소에 저장하고 ArgoCD와 같은 도구가 이를 지속적으로 모니터링하여 클러스터의
실제 상태와 동기화

### 젠킨스의 쿠버네티스 매니페스트 업데이트 과정
젠킨스가 매니페스트 파일을 업데이트 하는 것은 매니페스트 파일 내부의 이미지 파일만 업데이트 한다.
기존 매니패스트에 젠킨스가 새로 생성한 이미지 태그만 변경하고 다른 구성요소는 그대로 유지

**젠킨스가 파이프라인에 따라 새로운 이미지를 생성하면 해당 이미지의 태그만 기존 매니페스트 파일에서 수정**


## Jenkins 최종 정리

Jenkins는 CI(지속적 통합)도구로서 개발자가 소스 코드를 코드 저장소에 업데이트를 하면 이를 감지해서
빌드 > 테스트 > 이미지 생성 > 이미지 레지스트리 푸시 > 매니페스트 업데이트를 자동으로 수행한다.

빌드, 테스트 뿐만 아니라 컨테이너 기술에서 사용되는 도커/컨테이너 이미지를 생성하고
이를 다른 계층에서 사용할 수 있도록 이미지 레지스트리에 푸시하는 역할도 한다.

또한 생성한 이미지의 태그를 쿠버네티스 매니페스트에 업데이트해서 쿠버네티스가 배포할 때 사용할 최신의
이미지를 제공하는 역할도 한다.
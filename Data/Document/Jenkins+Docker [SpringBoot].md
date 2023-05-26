[뒤로가기](../../README.md)<br>

# Jenkins + Docker [SpringBoot]

Webhook을 통해 Github의 프로젝트를 Jenkins로 자동 빌드를 하고,
빌드한 Jar파일을 도커 이미지로 빌드한다.<br>
그리고 생성된 도커 이미지를 DockerHub에 Push하고 DockerHub에 <br>올라간 도커 이미지를 Pull받아 컨테이너로 만든뒤 서버 배포<br>

## 구성도

![img](../Img/jenkins%2BDocker%20%5Bspringboot%5D.png)

## 사전 세팅

1. AWS EC2 Ubuntu 생성 및 환경 설정 [AWS EC2 생성](../Document/EC2.md)
2. Springboot 프로젝트 생성 [SpringBoot 프로젝트 생성](../Document/Local%20%5BSpringBoot%5D.md)
3. SpringBoot 프로젝트 Github에 저장
4. Docker 설치 [Dokcer](../Document/Docker.md)

## 환경

1. Jenkins 서버용 EC2 [Java SE-17](https://www.oracle.com/java/technologies/javase/jdk17-archive-downloads.html)<br>
2. WAS 서버용 EC2
3. VSCode<br>
   (1) Spring Boot Extension Pack<br>
   (2) Extension Pack for Java<br>
   (3) Gradle for Java<br>

### [권장]<br>

[Jenkins 서버 swap메모리 할당](../Document/Linux.md)

## 시작

## Jenkins 서버 환경 구축

### (1) Dokcer 설치

생략 - [Docker 설치](../Document/Docker.md)<br>

### (2) Jenkins 이미지 Pull 및 컨테이너 실행

![img](../Img/docker%2Bjenkins1.png)<br>

> docker pull jenkins/jenkins<br>

![img](../Img/docker%2Bjenkins2.png)<br>

> docker run --privileged -d -p 8080:8080 -p 50000:50000 --name jenkins jenkins/jenkins<br>

(jenkins 이미지를 컨테이너로 넣어 실행)<br>
여기서 --privileged는 시스템의 주요 자원에 접근 할 수 있게 권한을 설정함<br>

### (3) Jenkins 접속 확인 및 환경 설정

![img](../Img/docker%2Bjenkins3.png)<br>

![img](../Img/docker%2Bjenkins4.png)<br>

> docker exec -itu 0 jenkins /bin/bash<br>

(root권한으로 jenkins 이미지에 접근)<br>

![img](../Img/docker%2Bjenkins5.png)<br>

> cat /var/jenkins_home/secrets/initialAdminPassword<br>

(초기 비밀번호를 복사해 로그인)<br>
![img](../Img/docker%2Bjenkins6.png)<br>
(jeknins가 제안하는 플러그인 설치)<br>

---

### [주의사항]

실습에 쓰이는 springboot 프로젝트는 현재 java17을 지원함<br>
하지만 jenkins의 java 버전이 11이므로 17로 변경을 해줘야함<br>
(본인의 프로젝트 자바 버전에 맞춰주세요.)<br>

Java 17로 설정하기<br>

![img](../Img/docker%2Bjenkins7.png)<br>

> apt-get update<br>

(일단 Jenkins 이미지의 패키지를 업데이트 해줌)<br>

![img](../Img/docker%2Bjenkins8.png)<br>

> apt-get install openjdk-17-jdk -y<br>

(openjdk17 버전 설치)<br>

![img](../Img/docker%2Bjenkins9.png)<br>

> export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64<br>
> echo $JAVA_HOME<br>

(환경 변수 변경 및 확인 <br>
JAVA_HOME 경로 복사)<br>

![img](../Img/docker%2Bjenkins10.png)<br>
(jenkins괸리 -> Tools로 이동)

![img](../Img/docker%2Bjenkins11.png)<br>
(JDK installations에서 Add JDK를 누르고 다음과 같이 설정후 저장)<br>

---

### (3) Jenkins 와 Github 연동 및 Webhook 설정

github에 저장된 본인 프로젝트와 Jenkins 연동 및 Webhook 설정은
[Jenkins [SpringBoot]](../Document/Jenkins%20%5BSpringBooot%5D.md)를 참고<br>

[참고]<br>

### [1] SSH 키 경로

![img](../Img/docker%2Bjenkins12.png)<br>

> ssh-keygen ssh-keygen -t rsa<br>

(키를 현재 경로에서 생성한 뒤)<br>

![img](../Img/docker%2Bjenkins13.png)<br>

(해당 경로로 가면 생성된 키가 있음)<br>

### [2] 플러그인 설치 시 jenkins가 꺼짐 현상

![img](../Img/docker%2Bjenkins14.png)<br>

(플러그인을 설치한뒤 restart를 해도 jenkins가 재실행이 안됨)<br>

![img](../Img/docker%2Bjenkins15.png)<br>

> docker start jenkins<br>

(start 명령어로 실행하면 됨)<br>

### (4) Github 연동 테스트

연동 테스트를 진행하기 전에
Springboot 프로젝트에 Dockerfile을 추가하도록 하겠음<br>

![img](../Img/docker%2Bjenkins16.png)<br>

> From openjdk:17-oracle<br>
> ARG JAR_FILE=build/libs/jenkinsdemo.jar<br>
> ADD ${JAR_FILE} jenkinsdemo.jar<br>
> ENTRYPOINT ["java","-jar","/jenkinsdemo.jar"]<br>

(jar을 파일명은 본인 프로젝트 jar파일 명으로 수정<br>
저장하고 git 커밋)<br>

![img](../Img/docker%2Bjenkins17.png)<br>
(빌드는 진행하지 않지만 Github과 연동이 되는것을 확인 할 수 있음)
<br>

### (5) Jenkins 컨테이너에 Docker 설치

Jenkis로 프로젝트를 빌드한뒤 Docker 이미지를 생성해
DockerHub에 저장<br>

![img](../Img/docker%2Bjenkins18.png)<br>

> apt-get update<br>

(패키지 업데이트)<br>

![img](../Img/docker%2Bjenkins19.png)<br>

> apt-get install \\<br>
> ca-certificates \\<br>
> curl \\<br>
> gnupg \\<br>
> lsb-release<br>

(필수 패키지 설치)<br>

![img](../Img/docker%2Bjenkins20.png)<br>

> mkdir -p /etc/apt/keyrings<br>

![img](../Img/docker%2Bjenkins21.png)<br>

curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \\<br>
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \\<br>
\$(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null<br>

(Docker 설치 환경 설정)<br>

![img](../Img/docker%2Bjenkins18.png)<br>

> apt-get update<br>

(패키지 업데이트)<br>

![img](../Img/docker%2Bjenkins22.png)<br>

> apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin<br>

(도커 설치)<br>

![img](../Img/docker%2Bjenkins23.png)<br>

> service docker start<br>

(도커 실행)<br>

![img](../Img/docker%2Bjenkins24.png)<br>

> usermod -aG docker root<br>
> su - root<br>

(Docker 그룹에 root 계정 추가)<br>

![img](../Img/docker%2Bjenkins25.png)<br>

> chmod 666 /var/run/docker.sock<br>

(docker.sock 권한 변경)<br>

![img](../Img/docker%2Bjenkins26.png)<br>

> docker login<br>

(docker 로그인 [본인 계정으로 로그인])<br>

### (6) Jenkins publish over ssh 설정

publish over ssh 설치와 설정은 [Jenkins [SpringBoot]](../Document/Jenkins%20%5BSpringBooot%5D.md)를 참고

### [빌드 관련]<br>

![img](../Img/docker%2Bjenkins27.png)<br>

Build steps에서<br><br>
chmod +x gradlew<br>
./gradlew clean build<br>
를 입력해 프로젝트를 빌드 설정<br>

docker login -u 본인유저네임 -p 본인비밀번호 docker.io<br>
docker build -t 본인유저네임/레파지토리명 .<br>
docker push 본인유저네임/레파지토리명<br>
으로 설정<br>

[주의사항] 레파지토리명은 빌드파일명으로 하는것을 추천 (jenkinsdemo.jar)<br>

![img](../Img/docker%2Bjenkins28.png)<br>

(빌드후 조치에서 Send build artifacts over SSH을 선택)<br>

docker login -u 본인유저네임 -p 본인비밀번호 docker.io<br>
docker pull 본인유저네임/레파지토리명<br>
docker ps -q --filter name=컨테이너명 | grep -q . && docker rm -f $(docker ps -aq --filter name=컨테이너명)<br>
docker run -d --name 컨테이너명 -p 8080:8080 본인유저네임/레파지토리명<br>

(참고로 filter는 실행중인 컨테이너가 있으면 해당 컨테이너를 중지하고 삭제하는 것임)<br>

(이상태로 저장)<br>

### (7) DockerHub 레파지토리 생성

![img](../Img/docker%2Bjenkins29.png)<br>
(Dockerhub에 접속해 본인의 레파지토리 생성)
[Dockerhub 생성법](../Document/Docker.md)<br>

### (8) was 서버 Docker 설치

[Docker 설치](../Document/Docker.md)<br>

[주의사항]<br>
![img](../Img/docker%2Bjenkins30.png)<br>

> sudo usermod -aG docker $USER<br>
> sudo su - $USER<br>

(docker 그룹에 계정을 추가)<br>

### (9) 빌드 테스트

![img](../Img/docker%2Bjenkins31.png)<br>
(서버 배포가 된것을 확인)<br>

### (10) 최종 테스트

![img](../Img/docker%2Bjenkins32.png)<br>
(프로젝트를 수정한뒤 Github에 커밋)<br>

![img](../Img/docker%2Bjenkins33.png)<br>
(자동으로 WAS의 컨테이너를 삭제하고 배포 하는것을 볼 수 있음)<br>

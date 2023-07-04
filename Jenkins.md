# Jenkins

[뒤로가기](./)\


## Jenkins

지속적인 통합(CI) , 지속적인 배포(CD) 환경을 구축하기 위한 도구\
빌드, 테스트, 배포 프로세스를 자동화하여 소프트웨어 품질과 개발 생산성을 높일 수 있다.\


### 설치

#### 환경

1. 우분투 22.04
2. java 11

#### (1) JDK 설치

![img](Data/Img/jenkins1.png)\
(현재 jenkins는 java 11버전이 필요하다.)\


#### (2) Jenkins 저장소 key 등록

![img](Data/Img/jenkins2.png)\
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \\\
/usr/share/keyrings/jenkins-keyring.asc > /dev/null

#### (3) apt 저장소에 추가

![img](Data/Img/jenkins3.png)\
echo deb \[signed-by=/usr/share/keyrings/jenkins-keyring.asc] \\\
https://pkg.jenkins.io/debian binary/ | sudo tee \\\
/etc/apt/sources.list.d/jenkins.list > /dev/null

#### (4) 패키지 업데이트

![img](Data/Img/jenkins4.png)\


#### (5) jenkins 설치

![img](Data/Img/jenkins5.png)\


\[참고] [설치가 느릴겨우](etc/Linux.md)

#### (6) 설치 확인

![img](Data/Img/jenkins6.png)\


#### (7) 접속 및 환경 설정

![img](Data/Img/jenkins7.png)\
(젠킨스 서버 포트는 기본적으로 8080이다.\
자신의 IP:8080 으로 접속하자.\
그리고 다음과 같은 화면에서\
/var/lib/jenkins/secrets/initialAdminPassword 해당 줄을 복사한다.)\


![img](Data/Img/jenkins8.png)\
(파일의 내용을 출력하면 다음과 같은 암호가 나온다.\
해당 암호를 방금전 사이트에 복사해서 붙여넣는다. )\


![img](Data/Img/jenkins9.png)\


![img](Data/Img/jenkins10.png)\
(플러그인 설치 화면이다. 일단, 추천하는 플러그인으로 설치한다.)\


![img](Data/Img/jenkins11.png)\
(설치가 완료되면 계정을 생성해준다.)\


![img](Data/Img/jenkins12.png)\
(젠킨스 URL 접속 주소이다. 기본으로 둔다.)\


![img](Data/Img/jenkins13.png)\
(설치 완료)\

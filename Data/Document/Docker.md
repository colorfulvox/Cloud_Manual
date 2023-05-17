[뒤로가기](../../README.md)

# 도커

컨테이너 기반 오픈소스 가상화 플랫폼 <br>
다양한 프로그램, 실행 환경을 컨테이너로 추상화하고 동일한 인터페이스를 제공하여 <br>프로그램의 배포 및 관리를 단순하게 해준다.<br>

가상 머신은 호스트 OS위에 게스트 OS 전체를 가상화 하여 사용하는 방식이다.
이 방식을 통해 HVM 가상화 기술도 등장했지만, <br>여전히 성능에 문제가 있다.<br>
그래서 이를 개선하기위해 프로세스 격리 방식을 선택하게 된다.

하나의 서버에 여러개의 컨테이너를 실행하면 서로 영향을 미치지 않고 독립적으로<br>
실행되어 마치 VM을 사용하는 느낌을 준다.

도커에서 가장 중요한 개념은 컨테이너, 이미지 이다.<br>
컨네이너 : 격리된 공간에서 프로세스가 동작하는 기술<br>
이미지 : 컨테이너 실행에 필요한 파일과 설정값을 포함한 것<br>

## 도커 설치

(1) 설치 <br>
curl -fsSL https://get.docker.com/ | sudo sh <br>
![img](../Img/docker1.png)<br>
![img](../Img/docker2.png)<br>
(설치 확인)<br>

(2) 명령어<br>
docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]<br>
(OPTIONS)<br>
![img](../Img/docker3.png)<br>

(3) 우분투 컨테이너 생성<br>
docker run ubuntu:16.04<br>
![img](../Img/docker4.png)
run 명령어는 사용할 이미지가 저장되어 있는지 확인하고 없다면 설치한 후<br>
컨테이너를 생성하고 시작해줌<br>
지금은 생성이 되었지만 별다른 명령어가 없어 자동으로 종료된다.<br>

(4) 컨테이너 실행<br>
docker run --rm -it ubuntu:16.04 /bin/bash<br>
![img](../Img/docker5.png)<br>

(5) centos 이미지 풀<br>
![img](../Img/docker6.png)<br>
![img](../Img/docker7.png)<br>
(현재 이미지 확인)<br>

(6) Dockerfile로 이미지 만들기<br>
![img](../Img/docker8.png)<br>
(폴더 생성 뒤 Dockerfile 생성한 후 vi 작성)<br>
![img](../Img/docker9.png)<br>
ubuntu:16.04 이미지를 생성하고 update한뒤 git을 설치하라<br>

![img](../Img/docker10.png)<br>
그 외 명령어<br>

docker ps // 실행중인 컨테이너 목록<br>
docker images //이미지 확인<br>
/- docker 그룹에 사용자 추가 -/<br>
sudo usermod -aG docker $USER<br>
sudo su - $USER<br>
/---------------------------------/<br>
docker rm 컨테이너 tag or id //컨테이너 삭제
dokcer rmi tag or id //이미지 삭제

## 도커 허브

자신이 만든 도커 이미지를 저장할 수 있는 클라우드 저장소<br>

### [도커허브](https://hub.docker.com/)

(1) 회원가입 생략<br>
(2) 저장소 생성<br>
![img](../Img/hub1.png)<br>
![img](../Img/hub2.png)<br>
![img](../Img/hub3.png)<br>

(3) 도커 이미지 업로드<br>
![img](../Img/hub4.png)<br>
(docker 로그인)<br>
![img](../Img/hub5.png)<br>
(반드시 빌드시 이미지 이름을 유저이름/레파지토리:태그 형식으로 저장한다.)<br>
![img](../Img/hub6.png)<br>
(저장소에 해당 이미지 Push)<br><br>
![img](../Img/hub7.png)<br>

## [Docker를 활용한 SpingBoot 배포](./GithubActions%2BDokcer%2BSpringboot.md)

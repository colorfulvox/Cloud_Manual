[뒤로가기](../../README.md)

# Local [SpringBoot]

LocalPC에서 SpringBoot 프로젝트를 빌드한뒤 해당 파일을 EC2에 전송시켜 서버를 배포하는 방식

## 구성도

![img](../Img/local%2BSpringBoot.png)

## 환경

[Java SE-17](https://www.oracle.com/java/technologies/javase/jdk17-archive-downloads.html)<br>
VSCode

1. Spring Boot Extension Pack
2. Extension Pack for Java
3. Gradle for Java

EC2 - Ubuntu (AWS EC2 생성 과정 참조)
[EC2](../Document/EC2.md)

## 시작

### (1) SpringBoot 프로젝트 생성

![img](../Img/local1.png)<br>
![img](../Img/local2.png)<br>
(Create a Gradle Project로 선택)<br>
다음은 이 순서대로 세팅<br>
3.0.7 -> Java -> com.example -> demo -> Jar -> 17 -> Spring Web (해당 항목만 체크하고 생성)<br><br>

[주의사항]
해당 위치에 Gradle Error 발생 알림이 뜨면 Vscode의 Java 환경 세팅 수정
![img](../Img/local3.png)<br><br>

![img](../Img/local4.png)<br>
(vscode File -> Preferences -> Settings 이동)<br><br>
![img](../Img/local5.png)<br>
(java home 검색 -> Edit in settings.json 클릭)<br><br>
![img](../Img/local6.png)<br>
(해당 항목들로 구성되어 있는지 확인)<br><br>

![img](../Img/local7.png)<br>
(src -> main -> resources -> static에 이동후 index.html 파일 생성)<br>
![img](../Img/local8.png)<br>
(index 파일에 !입력 후 엔터)
![img](../Img/local9.png)<br>
(body에 텍스트 작성후 저장)

![img](../Img/local10.png)<br>
(java -> DemoApplication.java 클릭하고 Run 클릭)
![img](../Img/local11.png)<br>
(웹사이트에서 localhost:8080 입력 후 정상 작동하는지 확인)

### (2) Gradle로 빌드후 Jar파일 생성

![img](../Img/local12.png)<br>
(코끼리 아이콘 클릭 -> Tasks -> build -> build에 실행 아이콘 클릭)<br><br>

[참고] 빌드 jar파일 이름 변경하기
![img](../Img/local13.png)<br>
(프로젝트 폴더의 build.gradle파일에 해당 항목을 추가한뒤 원하는 이름으로 수정)<br><br>

![img](../Img/local14.png)<br>
(빌드가 완료되면 프로젝트 폴더에 build -> libs -> 원하는이름.jar 파일이 생성된 것을 확인)<br>

![img](../Img/local15.png)<br>
(해당 파일을 오른쪽 클릭 한뒤 Reveal in File Explorer을 클릭해 해당 폴더 열기)
![img](../Img/local16.png)<br>
(폴더가 열리면 오른쪽 클릭 후 터미널 열기<br>
만약 터미널이 없을 경우
CMD를 열어 "cd 폴더 경로" 를 입력해 이동)<br>
![img](../Img/local17.png)<br>
![img](../Img/local11.png)<br>
(java -jar 해당파일.jar 를 입력 후 실행한뒤 다시 웹사이트에서 localhost:8080을 입력해 작동하는지 확인)<br>

### (3) EC2 환경 세팅

(3_1) 탄력적 IP 할당<br>
인스턴스의 Public IP는 고정 IP가 아닌 유동적인 IP이다.<br>
(그래서 인스턴스를 중지하고 다시 실행하면 IP가 바뀐다.)<br>
그래서 고정된 IP를 생성해 인스턴스에 연결해준다.<br><br>
![img](../Img/local18.png)<br>
(AWS로 가서 탄력적 IP 클릭)<br>
![img](../Img/local19.png)<br>
(탄력적 IP 주소 할당 클릭)<br>
![img](../Img/local20.png)<br>
(할당 클릭)<br>
![img](../Img/local21.png)<br>
(할당된 IP 클릭)<br>
![img](../Img/local22.png)<br>
(탄력적 IP 주소 연결 클릭)<br>
![img](../Img/local23.png)<br>
(연결할 인스턴스를 선택하고 연결 클릭)<br>
서버에 배포후 해당 IP주소로 접속 하면 됨 <br>

(3_2) 보안 그룹 생성<br>
SprintBoot의 기본 포트는 8080
하지만,
![img](../Img/local24.png)<br>
현재 인스턴스에 접속 가능한 포트는 22 밖에 없음<br>
그래서 8080포트를 생성 해야됨<br><br>
![img](../Img/local25.png)<br>
(보안 그룹에서 보안 그룹 생성 클릭)
![img](../Img/local26.png)<br>
(그룹 이름과 설명을 입력하고<br>규칙을 추가하면서
해당 항목에 맞게 구성하고 보안 그룹 생성 클릭)<br>
![img](../Img/local27.png)<br>
(인스턴스로 돌아가서 작업 -> 보안 -> 보안 그룹 변경)<br>
![img](../Img/local28.png)<br>
(생성한 보안 그룹을 선택한뒤 그룹 추가 후 저장 <br>[이미 연결된 보안 그룹은 제거해도 됨])<br>

![img](../Img/local29.png)<br>
(이렇게 구성되면 완료)<br>

### (4) 우분투 환경 세팅

![img](../Img/local30.png)<br>
(터미널로 우분투 접속)<br>

[참고]<br>
기본적으로 우분투 생성시<br>
sudo apt-get update<br>
sudo apt-get upgrade<br>
해주면 좋음<br><br>

![img](../Img/local31.png)<br>
(sudo apt install openjdk-17-jdk 을 입력 후
JDK 설치)<br>
![img](../Img/local32.png)<br>
(java -version으로 설치되었는지 확인)

### (5) 로컬에서 빌드한 jar 파일 우분투에 보내기

![img](../Img/local33.png)<br>
(생성된 jar파일을 pemkey파일이 있는 폴더로 이동한 뒤 터미널 실행)<br>

![img](../Img/local34.png)<br>
scp -i [pem파일경로] [업로드할 파일 이름] [ec2-user계정명]@[ec2 instance의 public DNS]:~/[경로]<br>
"[ec2-user계정명]@[ec2 instance의 public DNS]" 해당 부분은 인스턴스 연결의 SSH 클라이언트의 예시를 참고<br>
(해당 명령어에 맞춰 입력)<br>
![img](../Img/local35.png)<br>
(다시 우분투로 돌아가 ls 명령어로 옮겨졌는지 확인)<br>
![img](../Img/local36.png)<br>
(java -jar 파일이름.jar 명령어로 실행)<br>
![img](../Img/local37.png)<br>
(웹사이트로 가서 할당 받은 IP주소:8080으로 접속이 되는지 확인)<br>

[참고]<br>
우분투 터미널을 닫으면 서버가 중지되거나
다른 작업도 할 수 없는 상태임<br>
그래서 백그라운드로 해당 프로그램을 실행시켜 서버를 유지시켜야됨<br>

![img](../Img/local38.png)<br>
(nohup java -jar 파일이름.jar & 을 입력해
백그라운드 실행)<br>
[이제 터미널이 닫혀도 서버 접속이 가능하고 다른 작업도 가능함]

해당 프로그램을 종료하고 싶을 때<br>
![img](../Img/local39.png)<br>
(ps -ef | grep java로 실행되고 있는 프로그램의 PID를 확인 [11061] )<br>
![img](../Img/local40.png)<br>
(kill -9 해당PID 을 입력해 프로그램 종료)

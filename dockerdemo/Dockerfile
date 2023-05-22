From openjdk:17-oracle
# 도커 베이스 이미지

ARG JAR_FILE=build/libs/dockerdemo.jar
# ARG는 컨테이너 내에서 사용할 수 있는 변수를 지정할 수 있음
# 여기서는 JAR_FILE변수로 빌드 파일의 경로를 저장함

COPY ${JAR_FILE} app.jar
# JAR_FILE을 컨테이너의 app.jar로 복사

ENTRYPOINT ["java","-jar","/app.jar"]
# 컨테이너가 시작되었을때 실행되는 스크립트
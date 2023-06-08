[뒤로가기](../../README.md)<br>

# GithubActions+Docker [SpringBoot]

Github Actions가 프로젝트 소스를 받아 Docker 이미지로 빌드한뒤<br> DockerHub에 저장하고
저장한 이미지를 EC2에서 이미지를 실행해 배포<br>

## 구성도

![img](../Img/githubactions%2Bdocker.png)

## 사전 세팅

1. AWS EC2 Ubuntu 생성 및 환경 설정 [AWS EC2 생성](../Document/EC2.md)
2. Springboot 프로젝트 생성 [SpringBoot 프로젝트 생성](../Document/Local%20%5BSpringBoot%5D.md)
3. SpringBoot 프로젝트 Github에 저장
4. ubuntu에 도커 설치 [Docker 설치](../Document/Docker.md)

## 환경

[Java SE-17](https://www.oracle.com/java/technologies/javase/jdk17-archive-downloads.html)<br>
VSCode<br>

1. Spring Boot Extension Pack
2. Extension Pack for Java
3. Gradle for Java

EC2 - Ubuntu (AWS EC2 생성 과정 참조) [EC2](../Document/EC2.md)

## 시작

# S3-Bucket

[뒤로가기](../)

## S3 - 버킷 생성하기

![img](../Data/Img/s3\_1.png)\
(S3를 검색해 이동)\


![img](../Data/Img/s3\_2.png)\
(버킷 만들기 클릭)\


![img](../Data/Img/s3\_3.png)\
(일반 구성에서만 버킷 이름, AWS 리전을 선택한뒤 버킷 만들기 생성)\


![img](../Data/Img/s3\_4.png)\
(index.html 파일을 하나 만든다.)\


![img](../Data/Img/s3\_5.png)\
(생성한 버킷으로 들어가 파일을 끌어서 업로드 한다.)\


![img](../Data/Img/s3\_6.png)\
(업로드 클릭)\


![img](../Data/Img/s3\_7.png)\
(업로드가 잘된것을 확인한뒤 EC2를 생성한다.)\


\-> 그전에 EC2가 S3에 접근할 수 있도록 IAM에서 S3 접근권한을 부여하자\


![img](../Data/Img/s3\_8.png)\
(역할에서 역할 만들기 클릭)\


![img](../Data/Img/s3\_9.png)\
(EC2를 선택한뒤 다음)\


![img](../Data/Img/s3\_10.png)\
(S3를 검색한뒤 AmazonS3FullAccess를 선택한뒤 다음)\


![img](../Data/Img/s3\_11.png)\
(역할 이름만 정한뒤 역할 생성을 한다.)\


\-> 이제 EC2에서 S3에 접근할 수 있는 권한이 부여 되었으니 EC2를 생성하자.\


![img](../Data/Img/s3\_12.png)\
(EC2 생성시에 고급 세부 정보에서 방금 생성한 권한을 EC2에 부여해주자)\


![img](../Data/Img/s3\_13.png)\
(사용자 데이터에 해당 명령어를 입력한다. 단, S3의 경로는 생성한 버킷 이름으로 바꿔주자)

> \#!/bin/bash\
> yum install httpd\
> aws s3 cp s3://버킷이름/index.html /var/www/html --region ap-northeast-2\
> systemctl restart httpd\
>

\-> 그리고 EC2를 생성한다.\


![img](../Data/Img/s3\_14.png)\
(생성한 EC2를 접근하면 다음과같이 S3의 데이터를 받아와서 html을 배포하고 있는것을 볼 수 있다.)\

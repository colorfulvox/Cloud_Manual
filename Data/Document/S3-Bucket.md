[뒤로가기](../../README.md)

# S3 - 버킷 생성하기

![img](../Img/s3_1.png)<br>
(S3를 검색해 이동)<br>

![img](../Img/s3_2.png)<br>
(버킷 만들기 클릭)<br>

![img](../Img/s3_3.png)<br>
(일반 구성에서만 버킷 이름, AWS 리전을 선택한뒤 버킷 만들기 생성)<br>

![img](../Img/s3_4.png)<br>
(index.html 파일을 하나 만든다.)<br>

![img](../Img/s3_5.png)<br>
(생성한 버킷으로 들어가 파일을 끌어서 업로드 한다.)<br>

![img](../Img/s3_6.png)<br>
(업로드 클릭)<br>

![img](../Img/s3_7.png)<br>
(업로드가 잘된것을 확인한뒤 EC2를 생성한다.)<br>

-> 그전에 EC2가 S3에 접근할 수 있도록 IAM에서 S3 접근권한을 부여하자<br>

![img](../Img/s3_8.png)<br>
(역할에서 역할 만들기 클릭)<br>

![img](../Img/s3_9.png)<br>
(EC2를 선택한뒤 다음)<br>

![img](../Img/s3_10.png)<br>
(S3를 검색한뒤 AmazonS3FullAccess를 선택한뒤 다음)<br>

![img](../Img/s3_11.png)<br>
(역할 이름만 정한뒤 역할 생성을 한다.)<br>

-> 이제 EC2에서 S3에 접근할 수 있는 권한이 부여 되었으니 EC2를 생성하자.<br>

![img](../Img/s3_12.png)<br>
(EC2 생성시에 고급 세부 정보에서 방금 생성한 권한을 EC2에 부여해주자)<br>

![img](../Img/s3_13.png)<br>
(사용자 데이터에 해당 명령어를 입력한다. 단, S3의 경로는 생성한 버킷 이름으로 바꿔주자)

> #!/bin/bash<br>
> yum install httpd<br>
> aws s3 cp s3://버킷이름/index.html /var/www/html --region ap-northeast-2<br>
> systemctl restart httpd<br>

-> 그리고 EC2를 생성한다.<br>

![img](../Img/s3_14.png)<br>
(생성한 EC2를 접근하면 다음과같이 S3의 데이터를 받아와서 html을 배포하고 있는것을 볼 수 있다.)<br>

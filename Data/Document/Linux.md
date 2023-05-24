[뒤로가기](../../README.md)<br>

# [패키지 설치가 느릴 경우]

저장소 설정 파일인 /etc/apt/sources.list를 수정해서
국내에서 제공하는 미러 사이트로 변경하여 속도를 높인다.

### (1) apt 저장소 변경<br>

![img](../Img/mirror1.png)<br>

> sudo vi /etc/apt/sources.list

sources.list 진입<br>

### (2) apt 저장소 변경<br>

![img](../Img/mirror2.png)<br>
:%s/ap-northeast-2.ec2.archive.ubuntu.com/mirror.kakao.com/ 을 입력후 엔터

### (3) sudo apt-get update<br><br>

# [EC2 프리 티어 속도 높이기]

> sudo dd if=/dev/zero of=/swapfile bs=128M count=32<br>
> sudo chmod 600 /swapfile<br>
> sudo mkswap /swapfile<br>
> sudo swapon /swapfile<br>
> sudo swapon -s<br>

> sudo vi /etc/fstab <br>

/swapfile swap swap defaults 0 0 을 마지막 줄에 입력<br>

> free

입력해 swapfile이 메모리를 받고 있는지 확인<br>

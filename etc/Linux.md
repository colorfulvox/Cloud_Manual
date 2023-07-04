# Linux

[뒤로가기](../)\


## \[패키지 설치가 느릴 경우]

저장소 설정 파일인 /etc/apt/sources.list를 수정해서 국내에서 제공하는 미러 사이트로 변경하여 속도를 높인다.

#### (1) apt 저장소 변경 

![img](../Data/Img/mirror1.png)\


> sudo vi /etc/apt/sources.list

sources.list 진입\


#### (2) apt 저장소 변경 

![img](../Data/Img/mirror2.png)\
:%s/ap-northeast-2.ec2.archive.ubuntu.com/mirror.kakao.com/ 을 입력후 엔터

#### (3) sudo apt-get update  

## \[EC2 프리 티어 속도 높이기]

> sudo dd if=/dev/zero of=/swapfile bs=128M count=32\
> sudo chmod 600 /swapfile\
> sudo mkswap /swapfile\
> sudo swapon /swapfile\
> sudo swapon -s\
>

> sudo vi /etc/fstab\
>

/swapfile swap swap defaults 0 0 을 마지막 줄에 입력\


> free

입력해 swapfile이 메모리를 받고 있는지 확인\

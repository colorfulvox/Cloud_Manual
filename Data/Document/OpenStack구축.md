<<<<<<< HEAD
# OpenStack구축

[뒤로가기](../../)\


## \[OpenStack 구축]

### 시스템 구성도

![img](../Img/openstack38.png)\
오픈스택 공식 매뉴얼에서 제안하는 하드웨어 구성안이다.\


Controller Node는 오픈스택 서비스를 관리하는 노드로\
네트워크 타임 프로토콜, 데이터베이스 등 각 서비스의 서버에 해당하는 컴포넌트를 설치한다.\
여기서 NIC(네트워크 인터페이스 카드)는 관리용 1개, 외부용 1개로 구성된다.\


Compute Node는 가상 서버를 생성할 수 있는 하이퍼바이저가 설치되는 노드로 즉, 인스턴스이다.\


Block Storage Node는 인스턴스에 연결되는 하드 디스크를\
생성하고 관리하는 노드로 오픈스택 서비스 중 Cinder를 설치한다.\
기본적으로 LVM을 사용하며 인스턴스와 Compute Node, Controller Node를\
서로 연동하므로 관리용 NIC가 1개 필요하다.\


Object Storage Node는 파일을 관리하는 노드로 오픈스택 서비스중 Swift를 설치한다.\


### 환경

* Openstack : yoga
* Controller Node
  * VirtualBox
  * CPU : 2
  * Memory : 4096MB
  * Network Device : Host(enp0s3), Bridge(enp0s8), NAT(enp0s9)
  * Storage : SATA 20GB(운영체제)
  * Ubuntu : ubuntu-20.04.6-live-server-amd64
* Compute Node
  * VirtualBox
  * CPU : 2
  * Memory : 5012MB
  * Network Device : Host(enp0s3), Bridge(enp0s8), NAT(enp0s9)
  * Storage : SATA 20GB(운영체제), 8GB(블록 Storage), 8GB \* 3 (오브젝트 Storage)
  * Ubuntu : ubuntu-20.04.6-live-server-amd64

### [가상 머신 생성 및 환경 설정](<가상 머신 생성 및 환경 설정.md>)

### [Openstack패키지 설치 및 DB 구성](<Openstack패키지 설치 및 DB 구성.md>)

### [Openstack 메시지 큐 RabbitMQ 설치](<Openstack 메시지 큐 RabbitMQ 설치.md>)

### [인증서비스 KeyStone 설치](Openstack\_keystone.md)
=======
[뒤로가기](../../README.md)<br>

# [OpenStack 구축]

## 시스템 구성도

![img](../Img/openstack38.png)<br>
오픈스택 공식 매뉴얼에서 제안하는 하드웨어 구성안이다.<br>

Controller Node는 오픈스택 서비스를 관리하는 노드로 <br>네트워크 타임 프로토콜, 데이터베이스 등 각 서비스의 서버에 해당하는 컴포넌트를 설치한다.<br>
여기서 NIC(네트워크 인터페이스 카드)는 관리용 1개, 외부용 1개로 구성된다.<br>

Compute Node는 가상 서버를 생성할 수 있는 하이퍼바이저가
설치되는 노드로 즉, 인스턴스이다.<br>

Block Storage Node는 인스턴스에 연결되는 하드 디스크를 <br>생성하고 관리하는 노드로 오픈스택 서비스 중 Cinder를 설치한다.<br>
기본적으로 LVM을 사용하며 인스턴스와 Compute Node, Controller Node를<br> 서로 연동하므로 관리용 NIC가 1개 필요하다.<br>

Object Storage Node는 파일을 관리하는 노드로 오픈스택 서비스중 Swift를 설치한다.<br>

## 환경

- Openstack : yoga
- Controller Node

  - VirtualBox
  - CPU : 2
  - Memory : 4096MB
  - Network Device : Host(enp0s3), Bridge(enp0s8), NAT(enp0s9)
  - Storage : SATA 20GB(운영체제)
  - Ubuntu : ubuntu-20.04.6-live-server-amd64

- Compute Node
  - VirtualBox
  - CPU : 2
  - Memory : 5012MB
  - Network Device : Host(enp0s3), Bridge(enp0s8), NAT(enp0s9)
  - Storage : SATA 20GB(운영체제), 8GB(블록 Storage), 8GB \* 3 (오브젝트 Storage)
  - Ubuntu : ubuntu-20.04.6-live-server-amd64

## [가상 머신 생성 및 환경 설정](../Document/%EA%B0%80%EC%83%81%20%EB%A8%B8%EC%8B%A0%20%EC%83%9D%EC%84%B1%20%EB%B0%8F%20%ED%99%98%EA%B2%BD%20%EC%84%A4%EC%A0%95.md)

## [Openstack패키지 설치 및 DB 구성](../Document/Openstack%ED%8C%A8%ED%82%A4%EC%A7%80%20%EC%84%A4%EC%B9%98%20%EB%B0%8F%20DB%20%EA%B5%AC%EC%84%B1.md)

## [Openstack 메시지 큐 RabbitMQ 설치](../Document/Openstack%20%EB%A9%94%EC%8B%9C%EC%A7%80%20%ED%81%90%20RabbitMQ%20%EC%84%A4%EC%B9%98.md)

## [인증서비스 KeyStone 설치](../Document/Openstack_keystone.md)
>>>>>>> parent of 4d4a984 (GITBOOK-5: change request with no subject merged in GitBook)

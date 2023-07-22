[뒤로가기](../../README.md)<br>

# 컴퓨팅 자원 관리 서비스 Placement

VM 인스턴스 및 다른 리소스를 배치하고 관리하기 위해 사용된다.<br>
RAM, vCPU, 공유 스토리지, IP 등의 pool을 추적, 관리하며<br> 각 리소스에 대한 표준된 단위로 관리한다.<br>
Nova, neutron, cinder 등 컴퓨팅 서비스와 관련된
서비스를 구성하기전에 설치한다.<br>

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

## 시작

### (1) placement DB 생성 [Controller]

![img](../Img/openstack_169.png)

```
mysql -u root -popenstack

CREATE DATABASE placement;

GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'localhost' IDENTIFIED BY 'placementdbpass';

GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'%' IDENTIFIED BY 'placementdbpass';

exit;
```

비밀번호는 placementdbpass 로 설정한다.

### (2) Keystone인증을 위한 placement 계정 생성 [Controller]

![img](../Img/openstack_170.png)

> . adminrc.sh

관리자로 접속한다.<br>

![img](../Img/openstack_171.png)<br>

> openstack user create --domain default --password-prompt placement

placement 계정을 생성한다.<br>
비밀번호는 placementpass로 설정한다.<br>

![img](../Img/openstack_172.png)<br>

> openstack role add --project service --user placement admin

관리자 권한도 부여한다.<br>

![img](../Img/openstack_173.png)<br>

> openstack service create --name placement --description "Placement API" placement

placement 서비스를 생성한다.<br>

![img](../Img/openstack_174.png)<br>

```
openstack endpoint create --region RegionOne placement public http://controller:8778

openstack endpoint create --region RegionOne placement internal http://controller:8778

openstack endpoint create --region RegionOne placement admin http://controller:8778
```

EndPoint URL를 설정한다.<br>

### (3) placement 패키지 설치 및 설정 [Controller]

![img](../Img/openstack_175.png)<br>

> apt install placement-api

placement를 설치한다.<br>

![img](../Img/openstack_176.png)<br>

> vi /etc/placement/placement.conf

placement 구성 파일에 들어간다.<br>

![img](../Img/openstack_177.png)<br>

> auth_strategy = keystone

[api] 섹션에서 placement의 인증 방식을 keystone으로 설정한다.

![img](../Img/openstack_178.png)<br>

```
auth_url = http://controller:5000/v3
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = placement
password = placementpass
```

[keystone_authtoken] 섹션에서 keystone 설정도 한다.

![img](../Img/openstack_179.png)<br>

> connection = mysql+pymysql://placement:placementdbpass@controller/placement

[placement_database] 섹션에서 DB를 설정한뒤 저장한다.

![img](../Img/openstack_180.png)<br>

> su -s /bin/sh -c "placement-manage db sync" placement

db와 연동을 진행한다.

### (4) 설치 확인 [Controller]

![img](../Img/openstack_181.png)<br>

> service apache2 restart<br>
> service apache2 status

apache2를 재시작하고 정상 작동하는지 확인한다.<br>

![img](../Img/openstack_170.png)

> . adminrc.sh

관리자로 접속한다.<br>

![img](../Img/openstack_182.png)<br>

> placement-status upgrade check

Nova의 placement 서비스에 대한 업그레이드 상태를 체크한다.<br>
placement의 DB 구성 요소의 변경 사항을 검사하여 업그레이드 진행 전에 확인할 수 있다.<br>
정상적으로 나오면 설치가 완료되었다.<br>

### (5) placement 확장 모듈 [Controller]

![img](../Img/openstack_183.png)<br>

> pip3 install osc-placement

osc-placement는 openstack 서비스와 상호 작용하기 위한<br> 커맨드라인 인터페이스 도구이다.<br>
해당 모듈로 Compute 노드의 리소스 정보 조회 및 VM 인스턴스를<br> 배치하기위한 호스트를 선택할 수 있다.<br>

![img](../Img/openstack_184.png)<br>

> openstack --os-placement-api-version 1.2 resource class list --sort-column name

Placement 서비스에서 리소스 클래스를 조회하는 명령어다.<br>
리소스 클래스는 가용한 리소스 유형을 식별하는데 사용하고<br> VM인스턴스에 할당할 수 있는 CPU, 메모리, 디스크등과 같은 리소스를 표현한다.

## [참고]

### 각 서비스를 설치한뒤 반드시 VM의 스냅샷을 찍어 기록해두자.

Openstack 설치 과정에서 에러가 날 경우 다시 복원할 수 있어야한다.<br>

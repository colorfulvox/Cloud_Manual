[뒤로가기](../../README.md)<br>

# 컴퓨트 서비스 Nova

Openstack의 메인 서비스인 Nova<br>
하이퍼바이저(VM)을 관리하고 VM과 연동해 인스턴스를 생성 및 관리한다.<br>
이미지를 Glance에서 가져오고 네트워크는 Neutron 서비스를 이용한다.<br>

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

### [Compute]

Nova-Compute : VM을 통해 가상 머신 인스턴스를 생성하고 관리하는 데몬<br> (KVM, QEMU)

## 시작

## [Controller]

### (1) Nova DB 생성 [Controller]

Nova도 다른 서비스와 같이 DB를 생성한다.<br>

![img](../Img/openstack_141.png)

> mysql -u root -popenstack<br>
> CREATE DATABASE nova_api;<br>
> CREATE DATABASE nova;<br>
> CREATE DATABASE nova_cell0;<br>

Nova-cells는<br>
cells기능은 분산된 방식으로 Compute 클라우드를 확장할 수 있다.<br>
해당 기능이 활성화되면 각 호스트는 셀이라는 그룹으로<br> 분할되어 관리하기 편해진다.<br>
각 셀은 독립적으로 동작하며 자체 DB를 가지게된다.<br>
그리고 여기서 nova_cell0 DB를 생성한다.<br>

![img](../Img/openstack_142.png)

```
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY 'novadbpass';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'novadbpass';

GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'novadbpass';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'novadbpass';

GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' IDENTIFIED BY 'novadbpass';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY 'novadbpass';
```

각 DB에 대한 액세스 권한을 부여한다.<br>
여기서는 비밀번호를 novadbpass로 한다.<br>

### (2) Keystone인증을 위한 Nova 계정 생성 [Controller]

![img](../Img/openstack_143.png)<br>

> . adminrc.sh

관리자로 접속한다.<br>

![img](../Img/openstack_144.png)<br>

> openstack user create --domain default --password-prompt nova

Nova 사용자를 생성한다.<br>
비밀번호는 novapass로 한다.<br>

![img](../Img/openstack_145.png)<br>

> openstack role add --project service --user nova admin

Nova 계정에 admin 권한을 부여한다.<br>

![img](../Img/openstack_146.png)<br>

> openstack service create --name nova --description "OpenStack Compute" compute

Nova 서비스를 생성한다.<br>

![img](../Img/openstack_147.png)<br>

> openstack endpoint create --region RegionOne compute public http://192.168.56.101:8774/v2.1<br>
> openstack endpoint create --region RegionOne compute public http://192.168.56.101:8774/v2.1<br>
> openstack endpoint create --region RegionOne compute admin http://192.168.56.101:8774/v2.1

Endpoint URL를 설정한다.<br>

### (3) Nova 패키지 설치 [Controller]

![img](../Img/openstack_148.png)<br>

> apt install nova-api nova-conductor nova-novncproxy nova-scheduler

Nova-api : Nova 서비스의 중심 역할을 담당하는 컴포넌트<br>
클라이언트와 상호 작용하기위한 RESTful API를 제공<br>

Nova-Conductor : Nova-Compute와 Nova의 DB간의 상호 작용을 중재<br>

Nova-novncproxy : VNC 연결을 통해 실행 중인 인스턴스와 연결하기 위한 프록시를 제공<br>

Nova-Scheduler : 대기열에서 VM의 인스턴스 요청을 받고<br>
실행중인 Compute Node 서버중에 호스트를 결정

### (4) Nova 환경 설정 [Controller]

![img](../Img/openstack_149.png)<br>

> vi /etc/nova/nova.conf

nova 구성 파일에 들어간다.<br>

![img](../Img/openstack_150.png)<br>

> connection = mysql+pymysql://nova:novadbpass@192.168.56.101/nova_api

[api_database] 섹션에 입력한다.

![img](../Img/openstack_151.png)<br>

> connection = mysql+pymysql://nova:novadbpass@192.168.56.101/nova

[database] 섹션에 입력한다.

![img](../Img/openstack_152.png)<br>

> transport_url = rabbit://openstack:rabbitpass@192.168.56.101:5672/<br>
> my_ip = 192.168.56.101

맨 상단의 [Default] 섹션에 rabbit도 연동한다.

![img](../Img/openstack_153.png)<br>

```
www_authenticate_uri = http://192.168.56.101:5000/
auth_url = http://192.168.56.101:5000/
memcached_servers = 192.168.56.101:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = nova
password = novapass
```

[keystone_authtoken] 섹션에서 keystone 설정을 한다.

![img](../Img/openstack_154.png)<br>

> enabled = true<br>
> server_listen = 192.168.56.101<br>
> server_proxyclient_address = 192.168.56.101<br>

[vnc] 섹션<br>
VNC는 Nova에서 VM 인스턴스의 그래픽 화면을 원격으로 접근하기위해 사용된다.<br>

enabled = true : VNC 기능을 활성화한다.<br>
server_listen : VNC 서버가 수신 대기할 IP 주소를 설정한다.<br>
server_proxyclient_address : VNC 프록시 클라이언트가 사용할 IP주소를 설정한다.<br>

VNC 프록시 클라이언트는 VNC 서버와 VNC 클라이언트 <br>사이에서 중개 역할을 수행하는 프로그램이다.<br>

![img](../Img/openstack_155.png)<br>

> api_servers = http://192.168.56.101:9292

[glance] 섹션에서<br>
이미지 관리 서비스인 glance의 서버 주소를 설정한다.

![img](../Img/openstack_156.png)<br>

> lock_path = /var/lib/nova/tmp

[oslo_concurrency] 섹션

![img](../Img/openstack_157.png)<br>

```
send_service_user_token = true
auth_url = https://192.168.56.101/identity
auth_strategy = keystone
auth_type = password
project_domain_name = Default
project_name = service
user_domain_name = Default
username = nova
password = novapass

```

[service_user] 섹션

![img](../Img/openstack_158.png)<br>

> auth_strategy = keystone

[api] 섹션

![img](../Img/openstack_185.png)<br>

```
region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://192.168.56.101:5000/v3
username = placement
password = placementpass
```

[placement] 섹션에서 placement 설정을 한다.

### (5) DB 연동 [Controller]

![img](../Img/openstack_159.png)<br>

> su -s /bin/sh -c "nova-manage api_db sync" nova

![img](../Img/openstack_160.png)<br>

> su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova

![img](../Img/openstack_161.png)<br>

> su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova

새로운 셀을 생성한다.

![img](../Img/openstack_162.png)<br>

> su -s /bin/sh -c "nova-manage db sync" nova

### (6) 설치 확인 [Controller]

![img](../Img/openstack_163.png)<br>

> su -s /bin/sh -c "nova-manage cell_v2 list_cells" nova

cello0과 cell1이 등록되었는지 확인한다.

![img](../Img/openstack_164.png)<br>

```
service nova-api restart
service nova-scheduler restart
service nova-conductor restart
service nova-novncproxy restart
```

![img](../Img/openstack_165.png)<br>
![img](../Img/openstack_166.png)<br>
![img](../Img/openstack_167.png)<br>
![img](../Img/openstack_168.png)<br>

```
service nova-api status
service nova-scheduler status
service nova-conductor status
service nova-novncproxy status
```

---

## [참고]

### 각 서비스를 설치한뒤 반드시 VM의 스냅샷을 찍어 기록해두자.

Openstack 설치 과정에서 에러가 날 경우 다시 복원할 수 있어야한다.<br>

---

## [Compute]

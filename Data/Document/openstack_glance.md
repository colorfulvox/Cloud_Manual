[뒤로가기](../../README.md)<br>

# 이미지 관리 서비스 Glance

인스턴스를 생성하려면 운영체제가 설치된 이미지가 필요하다.<br>
이런 이미지를 관리하는 서비스가 Glance이다.<br>
이미지의 기본 저장소는 Controller VM의 /var/lib/glance/images/에 저장되며<br> 환경 설정을 통해 오브젝트 스토리지 서비스인 Swift에도 저장할 수 있다.<br>

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

### (1) Glance DB 생성 [Controller]

Glance를 설치하기전에 Keystone 서비스와 같이 Glance DB를 생성한다.<br>

![img](../Img/openstack_123.png)<br>

```
mysql -u root -popenstack
CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'glancedbpass';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'glancedbpass';
exit;
```

과정은 Keystone과 비슷하다.<br>
glance에게 glancdDB의 사용권한을 주고 비밀번호는 glancedbpass로 설정한다.<br>

### (2) Keystone인증을 위한 Glance 계정 생성 [Controller]

![img](../Img/openstack_124.png)<br>

> . adminrc.sh

glance 계정을 생성하기 위해 admin으로 로그인한다.<br>

![img](../Img/openstack_125.png)<br>

> openstack user create --domain default --password-prompt glance

glance 계정을 생성한다.<br>
비밀번호는 glancepass로 설정한다.<br>

![img](../Img/openstack_126.png)<br>

> openstack role add --project service --user glance admin

glance 계정에 admin role을 부여한다.<br>

![img](../Img/openstack_127.png)<br>

> openstack service create --name glance --description "OpenStack Image" image

Glance 서비스를 생성한다.

### (3) Glance 엔드포인트 URL설정 [Controller]

![img](../Img/openstack_128.png)<br>

> openstack endpoint create --region RegionOne image public http://192.168.56.101:9292

> openstack endpoint create --region RegionOne image internal http://192.168.56.101:9292

> openstack endpoint create --region RegionOne image admin http://192.168.56.101:9292

Keystone을 생성할때는 bootstrap으로 엔드포인트를 생성했지만,<br>
CLI 방식으로는 다음과 같이 엔드포인트를 생성할 수 있다.<br>
여기서도 공개 URL, 내부 URL, 관리자URL을 생성했다.<br>

### (4) Glance 설치 및 환경 설정 [Controller]

![img](../Img/openstack_129.png)<br>

> apt-get install glance

Glance를 설치한다.<br>

![img](../Img/openstack_130.png)<br>

> vi /etc/glance/glance-api.conf

설치가 완료되면 glance 환경 설정 파일에 들어간다.<br>

![img](../Img/openstack_131.png)<br>

> connection = mysql+pymysql://glance:glancedbpass@192.168.56.101/glance

keystone과 같이 DB 연결 설정부터 한다.

![img](../Img/openstack_132.png)<br>

> stores = file,http<br>
> default_store = file<br>
> filesystem_store_datadir = /var/lib/glance/images/<br>

다음과 같이 입력한다.<br>

glance_store 부분은 이미지를 어떤 방식으로 저장하고<br>
어떤 디렉토리에 저장할지를 설정하는 곳이다.<br>

stores는 Glance에서 사용 가능한 이미지 저장소 유형을 지정한다.<br>
file은 파일 시스템에 이미지를 저장 방식으로<br>
filesystem_store_datadir = /var/lib/glance/images/<br> 해당 경로에 저장 될 수 있다.<br>
http는 이미지를 원격으로 다운로드해 사용하는 방식이다.<br>

default_store는 기본 저장 방식을 지정한다.<br>
여기서는 file 방식이 기본 저장 방식이다.<br>

![img](../Img/openstack_133.png)<br>

```
www_authenticate_uri = http://192.168.56.101:5000
auth_url = http://192.168.56.101:5000
memcached_servers = 192.168.56.101:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = glancepass
```

생성한 keystone에 대한 정보를 입력한다.<br>

![img](../Img/openstack_134.png)<br>

> flavor = keystone

또한, glance 서비스가 keystone을 사용하여 인증 및 접근 제어를 처리하게 설정한다.<br>

그리고 저장한다.<br>

![img](../Img/openstack_135.png)<br>

> su -s /bin/sh -c "glance-manage db_sync" glance

DB와의 동기화를 진행한다.<br>
다음과 같이 뜨면 정상적으로 완료된것이다.<br>

![img](../Img/openstack_136.png)<br>

> service glance-api restart<br>
> service glance-api status

glance를 재시작하고 정상적으로 작동하는지 확인한다.<br>

### (5) 이미지를 glance에 등록하기 [Controller]

![img](../Img/openstack_137.png)<br>

> . adminrc.sh

관리자로 로그인한다.<br>

![img](../Img/openstack_138.png)<br>

> wget http://download.cirros-cloud.net/0.4.0/cirros-0.4.0-x86_64-disk.img

해당 이미지를 다운받는다.<br>
cirros는 OpenStack 배포 테스트할때 도움이 되는 작은 Linux 이미지다.<br>

![img](../Img/openstack_139.png)<br>

```
glance image-create --name "cirros" \
--file cirros-0.4.0-x86_64-disk.img \
--disk-format qcow2 --container-format bare \
--visibility=public
```

glance에 해당 이미지를 등록한다.<br>

---

### 코드 설명

glance image-create --name : 생성할 이미지의 이름을 지정한다.<br>

--file cirros-0.4.0-x86_64-disk.img : 어떤 이미지를 등록할지 지정한다.<br>

--disk-format qcow2 :<br>
디크스 형식을 qcow2로 지정한다.<br>
qcow2는 QEMU Copy-On-Write 형식으로 가상 디스크 이미지를 <br>저장하기 위한 형식이다.<br>

--container-format bare :<br>
컨테이너 형식을 bare로 지정한다.<br>
bare는 이미지가 가상 머신의 디스크 이미지임을 나타낸다.<br>

--visibility=public : 이미지의 공개 여부이다.<br>
public으로 모든 사용자에게 공개된다.<br>

---

![img](../Img/openstack_140.png)<br>

> openstack image list

정상적으로 이미지가 등록 되었는지 확인한다.<br>
또한, /var/lib/glance/images에 해당 이미지가 저장되었는지 확인한다.<br>

## [참고]

### 각 서비스를 설치한뒤 반드시 VM의 스냅샷을 찍어 기록해두자.

Openstack 설치 과정에서 에러가 날 경우 다시 복원할 수 있어야한다.<br>

[뒤로가기](../../README.md)<br>

# 인증 서비스 Keystone

Openstack의 모든 서비스는 인증 서비스인 Keystone에 통과해야 이용할 수 있다.<br>
keystone은 주로 Controller에 설치한다.<br>

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

### (1) Memcached 설치 [Controller]

Keystone은 많은 사용자와 프로젝트에 대한 인증 정보를 관리하고 처리해야한다.<br>
그만큼 요청시마다 디스크를 액세스하며 응답 시간이 느려진다.<br>
캐시 시스템인 Memcached를 설치해서 디스크 액세스가 아닌 메모리 캐시 방식을 적용한다.<br>

![img](../Img/openstack_75.png)<br>

> apt-get install memcached python3-memcache

memcached를 설치한다.<br>

![img](../Img/openstack_76.png)<br>

> vi /etc/memcached.conf

memcached의 환경을 설정한다.<br>

![img](../Img/openstack_77.png)<br>

> -l 192.168.56.101

응답할 IP주소를 Controller IP로 수정한다.<br>

![img](../Img/openstack_78.png)<br>

> service memcached restart<br>
> service memcached status

재시작하고 정상적으로 실행하는지 확인한다.

### (2) Keystone DB 생성 [Controller]

Openstack의 서비스를 설치하기전 MariaDB에 서비스 관련 DB를 생성한다.<br>
(서비스의 구성과 데이터 관리를 원할하게 수행하기 위함)<br>

> mysql -u root -popenstack

mariaDB에 root사용자로 로그인한다.<br>
-p다음 비밀번호인 openstack을 입력하면 바로 로그인이 가능하다.<br>

![img](../Img/openstack_82.png)<br>

> CREATE DATABASE keystone;

keystone DB를 생성한다.

> GRANT ALL PRIVILEGES ON keystone.\* TO 'keystone'@'localhost' IDENTIFIED BY 'keystonedbpass';

keystone사용자에게 keystone DB에 대한 모든 권한과 localhost에 접속할 수 있게 한다.<br>
비밀번호는 keystonedbpass로 설정했다.<br>

> GRANT ALL PRIVILEGES ON keystone.\* TO 'keystone'@'%' IDENTIFIED BY 'keystonedbpass';<br>

keystone 사용자가 모든 호스트에서 접근할 수 있는 권한을 부여한다.<br>

> exit;

그리고 나간다.<br>

### (3) Keystone 설치 및 설정 [Controller]

![img](../Img/openstack_83.png)<br>

> apt-get install keystone

keystone 패키지를 설치한다.

![img](../Img/openstack_84.png)<br>

> vi /etc/keystone/keystone.conf

keystone 환경을 설정한다.<br>

![img](../Img/openstack_85.png)<br>

memcache_servers를 찾아 주석을 푼뒤 IP를 설정한다.<br>[cache] 항목에 있다.<br>

![img](../Img/openstack_86.png)<br>

> connection = mysql+pymysql://keystone:keystonedbpass@controller/keystone

아까 설정한 mariaDB에 접근할 수 있게 설정한다.<br>

![img](../Img/openstack_87.png)<br>

> provider = fernet<br>

주석을 풀고 저장한다.<br>

Token의 설정 방식을 fernet으로 설정한다.<br>
fernet : 대칭키 암호화 방식을 사용하며 Openstack의 보안을 강화하기 위해 도입된 암호화 기술<br>

![img](../Img/openstack_88.png)<br>

> su -s /bin/bash -c "keystone-manage db_sync" keystone

keystone DB와의 동기화를 진행한다.<br>
keystone 서비스의 DB가 초기화되고 필요한 테이블이 생성된다.<br>

![img](../Img/openstack_90.png)<br>

> cat /var/log/keystone/keystone-manage.log

제대로 수행했는지 확인하려면 keystone의 log를 확인하면 된다.<br>

![img](../Img/openstack_91.png)<br>

> keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone

keystone의 fernet token 암호화 시스템을 설정한다.<br>

![img](../Img/openstack_92.png)<br>

> cat /var/log/keystone/keystone-manage.log

이것도 log에서 확인할 수 있다.<br>

![img](../Img/openstack_93.png)<br>

> keystone-manage credential_setup --keystone-user keystone --keystone-group keystone

Keystone의 관리자 자격 증명을 설정한다.<br>
Keystone 관리자 자격 증명 파일을 생성 및
Keystone 관리자 사용자와 그룹을 설정한다.<br>

![img](../Img/openstack_96.png)<br>

> vi /etc/apache2/apache2.conf

apache 설정 파일을 연다.

![img](../Img/openstack_97.png)<br>

> ServerName controller

ServerName을 Controller IP로 설정한다.<br>

![img](../Img/openstack_98.png)<br>

> service apache2 restart<br>
> service apache2 status

apache를 다시 시작하고 실행하는지 확인한다.

### (4) Keystone 서비스 생성 [Controller]

여기까지 환경 설정을 마쳤다.<br>
이제 Openstack에 keystone 서비스를 생성한다.

![img](../Img/openstack_94.png)<br>

> export controller=192.168.56.101

"controller" 변수를 해당 IP 주소로 설정하여<br> 다른 곳에서 이 변수를 참조할 수 있게 한다.

![img](../Img/openstack_95.png)<br>

```
keystone-manage bootstrap --bootstrap-password 'adminpass' \
--bootstrap-admin-url http://controller:5000/v3/ \
--bootstrap-internal-url http://controller:5000/v3/ \
--bootstrap-public-url http://controller:5000/v3/ \
--bootstrap-region-id RegionOne
```

keystone 서비스 생성 및 엔드포인트 URL를 설정한다.<br>
ID 서비스는 일반적으로 사용자가 상호 작용하는 첫 번째 서비스이다.<br>
keystone은 사용자 인증 부분과 서비스 인증 부분을 관리한다.<br>

사용자일때<br>

1. 사용자 그룹인 Project
2. 사용자 계정 정보인 ID와 Password
3. 사용자 권한인 Role

서비스일때<br>

1. 서비스 등록
2. 엔드포인트 URL (관리자URL, 내부 URL, 공개 URL)

을 등록해야한다.<br>

RegionOne은 일반적으로 Openstack배포에 사용되는 지역이름이다.<br>
bootstrap은 keystone 서비스의 초기 구성 단계에서 사용되는 개념이다.<br>

### (5) 관리자 구성 [Controller]

Keystone 서비스는 생성했지만,<br>
이용할 수 있는 사용자가 없으니 admin 사용자를 구성한다.<br>

![img](../Img/openstack_99.png)<br>

> vi admin.sh

접속할 수 있는 관리자 계정 구성을 위해 bash쉘을 생성해 환경 변수로 등록한다.<br>

![img](../Img/openstack_100.png)<br>

```
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=adminpass
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
```

다음과 같이 입력하고 저장한다.

여기서

![img](../Img/openstack_101.png)<br>

> . admin.sh

sh을 실행시켜 admin으로 접속한다.<br>

![img](../Img/openstack_102.png)<br>

> openstack user list

유저 리스트에 admin이 등록되었는지 확인하다.<br>

![img](../Img/openstack_107.png)<br>

> openstack service list

keystone 서비스도 등록되었는지 확인한다.

### [경고 메시지 관련]

![img](../Img/openstack_103.png)<br>

다음과 같은 경고는 상위 버전의 cryptography 패키지에서 발생하는 경고이다.

![img](../Img/openstack_104.png)<br>

> pip3 uninstall matrix-synapse twisted cryptography bcrypt cftp

![img](../Img/openstack_105.png)<br>

> pip3 install cryptography==3.2

![img](../Img/openstack_106.png)<br>

이제 해당 경고가 안뜬다.

### (5) 유저 생성 [Controller]

admin 계정을 생성할때 bash shell로 등록했었다.<br>
이번에는 CLI 방식으로 Demo 일반 계정을 생성해본다.<br>

![img](../Img/openstack_108.png)<br>

> openstack project create --domain default --description "Service Project" service

서비스 프로젝트를 생성한다.<br>
서비스 프로젝트는 일반적으로 Openstack 서비스 컴포넌트를<br> 위한 별도의 프로젝트로 사용된다.<br>
그래서 Nova, Neutron, Cinder 등의 Openstack 서비스가 사용하는 리소스를<br> 관리하고 액세스하기 위해 생성한다.

왜 프로젝트를 생성하는가?<br>
프로젝트는 Openstack 환경에서 리소스의 구분과 격리를 제공한다.<br>
프로젝트는 독립된 네트워크, 가상 머신, 스토리지 등과 같은 리소스를 관리할 수 있다.<br>
이를 통해 리소스간의 충돌을 방지하며 팀이 독립적으로 리소스 사용 및 관리할 수 있다.<br>

![img](../Img/openstack_109.png)<br>

> openstack project create --domain default --description "Demo Project" demo

Demo project를 생성한다.

![img](../Img/openstack_110.png)<br>

> openstack user create --domain default --password-prompt demo

demo 계정을 생성한다.<br>
여기서 비밀번호는 demopass로 했다.<br>

![img](../Img/openstack_111.png)<br>

> openstack role create user

일반 사용자용 role이 없다.<br>
그래서 user라는 이름의 role을 생성한다.<br>

![img](../Img/openstack_112.png)<br>

> openstack role add --project demo --user demo user

user role에 demo 계정을 추가한다.

![img](../Img/openstack_113.png)<br>

> openstack role assignment list --user demo --project demo --names

해당 명령어로 demo계정이 user role에 등록되었는지 확인할 수 있다.<br>

### (6) Token 발급 [Controller]

Keystone을 통해서 Openstack 서비스를 이용하는 과정은 다음과 같다.

![img](../Img/openstack_114.png)<br>

1. User -> Keystone : 자격증명 요청(사용자 로그인)
2. Keystone -> User : 토큰 발급
3. User -> Nova : 토큰을 가지고 Nova에 접속 요청
4. Nova -> Keystone : User가 가져온 토큰이 제대로 된 토큰인지 Keystone에게 확인 요청
5. Nova -> Glance : Nova가 유저에게 받은 토큰을 가지고 이미지 요청
6. Glance -> Keystone : Glance가 Keystone에게 토큰 확인 요청
7. Glance -> Nova : 이미지를 보내줌
8. Nova -> Neutron : 토큰을 가지고 네트워크 요청
9. Neutron -> Keystone : Neutron이 Keystone에게 토큰 확인 요청
10. Nova -> Neutron : 확인된 유저로 접근 시도
11. Neutron -> Nova : 기존 Neutron이 갖고 있는 토큰과 Nova가 갖고 있는 토큰이 동일하면 접근 성공

그래서 유저의 토큰을 발급 받아본다.

![img](../Img/openstack_115.png)<br>

> . adminrc.sh

아까와 같이 환경 변수에 등록해 admin으로 로그인을 한다.

![img](../Img/openstack_116.png)<br>

> openstack token issue

해당 명령어로 토큰을 발급받을 수 있다.
Demo 계정도 받아보자.

![img](../Img/openstack_117.png)<br>

demorc.sh을 생성하고

```
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=demopass
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
```

다음과 같이 입력한다.

![img](../Img/openstack_118.png)<br>

> . demorc.sh

demo로 로그인 한뒤

![img](../Img/openstack_119.png)<br>

> openstack token issue

만들어지는지 확인한다.

### (7) Token 발급 [Compute]

![img](../Img/openstack_120.png)<br>

Compute VM에서도 token발행이 되는지 확인한다.<br>
다음과 같이 admin 환경 변수를 설정해 로그인한뒤<br>
Controller의 Keystone에게 요청해보자.<br>

![img](../Img/openstack_121.png)<br>

잘 받는것을 확인한다.<br>

![img](../Img/openstack_122.png)<br>

demo 계정도 잘 받아지는지 확인한다.<br>

## [참고]

### 각 서비스를 설치한뒤 반드시 VM의 스냅샷을 찍어 기록해두자.

Openstack 설치 과정에서 에러가 날 경우 다시 복원할 수 있어야한다.<br>

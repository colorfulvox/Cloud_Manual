# Openstack\_keystone

[뒤로가기](../)\


## 인증 서비스 Keystone

Openstack의 모든 서비스는 인증 서비스인 Keystone에 통과해야 이용할 수 있다.\
keystone은 주로 Controller에 설치한다.\


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

#### (1) Memcached 설치 \[Controller]

Keystone은 많은 사용자와 프로젝트에 대한 인증 정보를 관리하고 처리해야한다.\
그만큼 요청시마다 디스크를 액세스하며 응답 시간이 느려진다.\
캐시 시스템인 Memcached를 설치해서 디스크 액세스가 아닌 메모리 캐시 방식을 적용한다.\


![img](../Data/Img/openstack\_75.png)\


> apt-get install memcached python3-memcache

memcached를 설치한다.\


![img](../Data/Img/openstack\_76.png)\


> vi /etc/memcached.conf

memcached의 환경을 설정한다.\


![img](../Data/Img/openstack\_77.png)\


> \-l 192.168.56.101

응답할 IP주소를 Controller IP로 수정한다.\


![img](../Data/Img/openstack\_78.png)\


> service memcached restart\
> service memcached status

재시작하고 정상적으로 실행하는지 확인한다.

#### (2) MariaDB 설정 및 Keystone DB 생성 \[Controller]

Openstack의 서비스를 설치하기전 MariaDB에 서비스 관련 DB를 생성한다.\
(서비스의 구성과 데이터 관리를 원할하게 수행하기 위함)\


DB 생성전에 Openstack에 맞는 DB설정을 한다.\


![img](../Data/Img/openstack\_79.png)\


> cd /etc/mysql/mariadb.conf.d vi 50-server.cnf

mariadb설정 폴더에 들어간뒤 mariaDB를 설정 한다.\
50-server은 mariaDB 서버의 기본 설정을 정의하는 파일이다.\


![img](../Data/Img/openstack\_80.png)\


bind-address를 주석 처리한다.\
(mariaDB가 실행될때 50-server.cnf파일을 먼저 읽고\
openstack.cnf 파일의 bind-address를 무시해 주석 처리한다.)\


![img](../Data/Img/openstack\_89.png)\


> service mysql restart

mariaDB를 재시작 한다.\


![img](../Data/Img/openstack\_81.png)\
mariaDB에 root사용자로 로그인한다.\
\-p다음 비밀번호인 openstack을 입력하면 바로 로그인이 가능하다.\


![img](../Data/Img/openstack\_82.png)\


> CREATE DATABASE keystone;

keystone DB를 생성한다.

> GRANT ALL PRIVILEGES ON keystone.\* TO 'keystone'@'localhost' IDENTIFIED BY 'keystonedbpass';

keystone사용자에게 keystone DB에 대한 모든 권한과 localhost에 접속할 수 있게 한다.\
비밀번호는 keystonedbpass로 설정했다.\


> GRANT ALL PRIVILEGES ON keystone.\* TO 'keystone'@'%' IDENTIFIED BY 'keystonedbpass';\
>

keystone 사용자가 모든 호스트에서 접근할 수 있는 권한을 부여한다.\


> exit;

그리고 나간다.\


#### (3) Keystone 설치 및 설정 \[Controller]

![img](../Data/Img/openstack\_83.png)\


> apt-get install keystone apache2 libapache2-mod-wsgi-py3

keystone과 관련 패키지를 설치한다.

apache2 : 클라이언트의 요청을 처리하고 keystone 서비스를 실행하기 위해 Apache HTTP Server와의 통합이 필요하다.\


libapache2-mod-wsgi-py3 : Apache HTTP Server에서 Python 웹 애플리케이션을\
실행하기 위한 WSGI 모듈이다.\
Keystone의 Python 코드를 Apache HTTP Server와 통합하기 위해 사용된다.\


![img](../Data/Img/openstack\_84.png)\


> vi /etc/keystone/keystone.conf

keystone 환경을 설정한다.\


![img](../Data/Img/openstack\_85.png)\


memcache\_servers를 찾아 주석을 푼뒤 IP를 설정한다.\
\[cache] 항목에 있다.\


![img](../Data/Img/openstack\_86.png)\


> connection = mysql+pymysql://keystone:keystonedbpass@192.168.56.101/keystone

아까 설정한 mariaDB에 접근할 수 있게 설정한다.\


![img](../Data/Img/openstack\_87.png)\


> provider = fernet\
>

주석을 풀고 저장한다.\


Token의 설정 방식을 fernet으로 설정한다.\
fernet : 대칭키 암호화 방식을 사용하며 Openstack의 보안을 강화하기 위해 도입된 암호화 기술\


![img](../Data/Img/openstack\_88.png)\


> su -s /bin/bash keystone -c "keystone-manage db\_sync"

keystone DB와의 동기화를 진행한다.\
keystone 서비스의 DB가 초기화되고 필요한 테이블이 생성된다.\


![img](../Data/Img/openstack\_90.png)\


> cat /var/log/keystone/keystone-manage.log

제대로 수행했는지 확인하려면 keystone의 log를 확인하면 된다.\


![img](../Data/Img/openstack\_91.png)\


> keystone-manage fernet\_setup --keystone-user keystone --keystone-group keystone

keystone의 fernet token 암호화 시스템을 설정한다.\


![img](../Data/Img/openstack\_92.png)\


> cat /var/log/keystone/keystone-manage.log

이것도 log에서 확인할 수 있다.\


![img](../Data/Img/openstack\_93.png)\


> keystone-manage credential\_setup --keystone-user keystone --keystone-group keystone

Keystone의 관리자 자격 증명을 설정한다.\
Keystone 관리자 자격 증명 파일을 생성 및 Keystone 관리자 사용자와 그룹을 설정한다.\


![img](../Data/Img/openstack\_94.png)\


> export controller=192.168.56.101

"controller" 변수를 해당 IP 주소로 설정하여 다른 곳에서 이 변수를 참조할 수 있게 한다.

![img](../Data/Img/openstack\_95.png)\


> keystone-manage bootstrap --bootstrap-password 'keystonepass' \\\
> \--bootstrap-admin-url http://$controller:5000/v3/ \\\
> \--bootstrap-internal-url http://$controller:5000/v3/ \\\
> \--bootstrap-public-url http://$controller:5000/v3/ \\\
> \--bootstrap-region-id RegionOne

keystone 서비스의 시스템을 설정한다. 여기서 관리자URL, 내부 URL, 공개 URL을 설정한다.\
RegionOne은 일반적으로 Openstack배포에 사용되는 지역이름이다.\


![img](../Data/Img/openstack\_96.png)\


> vi /etc/apache2/apache2.conf

apache 설정 파일을 연다.

![img](../Data/Img/openstack\_97.png)\


> ServerName 192.168.56.101

ServerName을 Controller IP로 설정한다.\


![img](../Data/Img/openstack\_98.png)\


> service apache2 restart\
> service apache2 status

apache를 다시 시작하고 실행하는지 확인한다.

#### (4) 실행 테스트 \[Controller]

![img](../Data/Img/openstack\_99.png)\


> vi openrc.sh

Keystone 서비스를 등록하고 접속할 수 있는 엔드포인트 URL을 구성한다.\
여기서는 정상적으로 구동하는지 테스트만 진행한다.\


![img](../Data/Img/openstack\_100.png)\


> export OS\_PROJECT\_DOMAIN\_NAME=Default\
> export OS\_USER\_DOMAIN\_NAME=Default\
> export OS\_PROJECT\_NAME=admin\
> export OS\_USERNAME=admin\
> export OS\_PASSWORD=keystonepass\
> export OS\_AUTH\_URL=http://192.168.56.101:5000/v3\
> export OS\_IDENTITY\_API\_VERSION=3\
> export OS\_IMAGE\_API\_VERSION=2\
>

다음과 같이 입력하고 저장한다.

![img](../Data/Img/openstack\_101.png)\


> . openrc.sh

sh을 실행시켜 시스템에 적용한다.\


![img](../Data/Img/openstack\_102.png)\


다음과 같은 테이블이 나오면 성공이다.\


#### \[경고 메시지 관련]

![img](../Data/Img/openstack\_103.png)\


다음과 같은 경고는 상위 버전의 cryptography 패키지에서 발생하는 경고이다.

![img](../Data/Img/openstack\_104.png)\


> pip3 uninstall matrix-synapse twisted cryptography bcrypt cftp

![img](../Data/Img/openstack\_105.png)\


> pip3 install cryptography==3.2

![img](../Data/Img/openstack\_106.png)\


이제 해당 경고가 안뜬다.

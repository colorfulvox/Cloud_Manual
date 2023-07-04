[뒤로가기](../../README.md)<br>

# Devstack 설치

### 1. stack계정 생성

Devstack을 설치하려면 stack계정이 필요하다.<br>

![img](../Img/openstack27.png)<br>

> useradd -U -G sudo -s /bin/bash -m stack<br>

sudo 권한이 있는 stack 계정을 추가한다.<br>

> echo "stack ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers<br>

패스워드를 입력하지않고도 sudo를 사용할 수 있게 설정한다.<br>

> passwd stack<br>

stack계정의 패스워드를 설정한다.<br>

![img](../Img/openstack28.png)<br>

> su stack<br>
> cd<br>

stack 계정으로 전환한다.<br>

### 2. devstack 환경 설정 및 설치

![img](../Img/openstack29.png)<br>

> git clone https://git.openstack.org/openstack-dev/devstack<br>

git 명령어로 devstack를 다운받는다.<br>

![img](../Img/openstack30.png)<br>

> cd devstack/<br>
> vi local.conf<br>

devstack 폴더로 이동한뒤 local.conf를 연다.<br>
오픈스택의 기본 정보와 nova-network 서비스를 비활성화, Neutron 서비스를 활성화한다.<br>

![img](../Img/openstack31.png)<br>

> [[local|localrc]]<br>
> HOST_IP=192.168.56.103<br>
> SERVICE_HOST=192.168.56.103<br>
> MYSQL_HOST=192.168.56.103<br>
> RABBIT_HOST=192.168.56.103<br>
> GLANCE_HOSTPORT=192.168.56.103:9292<br>
>
> ADMIN_PASSWORD=openstack<br>
> DATABASE_PASSWORD=openstack<br>
> RABBIT_PASSWORD=openstack<br>
> SERVICE_PASSWORD=openstack<br>
>
> \# Nova-Network 비활성화<br>
> disable_service n-net<br>
>
> \# Neutron 활성화<br>
> enable_service neutron<br>
> enable_service q-svc<br>
> disable_service q-agt<br>
> enable_service q-dhcp<br>
> disable_service q-l3<br>
> enable_service q-meta<br>
>
> \# Neutron 네트워크 옵션<br>
>
> Q_USE_SECGROUP=True<br>
> FLOATING_RANGE="192.168.0.0/24"<br>
> FIXED_RANGE="10.0.0.0/24"<br>
> Q_FLOATING_ALLOCATION_POOL=start=192.168.0.100,end=192.168.0.200<br>
> PUBLIC_NETWORK_GATEWAY="192.168.0.1"<br>
> PUBLIC_INTERFACE=enp0s8<br>
>
> \# OpenVSwitch 프로바이더 네트워킹 환경설정<br>
>
> Q_USE_PROVIDERNET_FOR_PUBLIC=True<br>
> OVS_PHYSICAL_BRIDGE=br-enp0s8<br>
> PUBLIC_BRIDGE=br-enp0s8<br>
> OVS_BRIDGE_MAPPINGS=public:br-enp0s8<br>

다음과 같이 입력한다.<br>

FLOATING_RANGE는 Public IP 대역을 말하며 VM이 생성될때 처음 할당받는<br>
Private IP는 Fixed IP이며 동적으로 할당하여 외부 인터넷과 통신 할 수 있는 <br>IP가 FLOATING IP이다.

Q_USE_SECGROUP=True -> 보안 그룹 사용을 허용<br><br>
FLOATING_RANGE="192.168.0.0/24 -> 외부 네트워크로 사용될 enp0s8 IP범위 설정<br><br>
Q_FLOATING_ALLOCATION_POOL=start=192.168.0.100,end=192.168.0.200<br>-> FLOATING IP 범위에서 인스턴스에 할당할 IP범위를 설정<br><br>
PUBLIC_NETWORK_GATEWAY="192.168.0.1" <br>-> FLOATING IP 범위의 실제 물리 게이트웨이 보통 게이트웨이는 자신의 IP에서 마지막 값인 1로 설정한다.<br><br>
PUBLIC_INTERFACE=enp0s8 -> FLOATING IP가 사용할 실제 물리 네트워크 디바이스<br><br>
OVS_PHYSICAL_BRIDGE=br-enp0s8<br>
PUBLIC_BRIDGE=br-enp0s8<br>
OVS_BRIDGE_MAPPINGS=public:br-enp0s8<br>
-> 네트워크 브릿지 이름 설정<br>

![img](../Img/openstack32.png)<br>

> ./stack.sh

stack.sh을 실행해 devstack을 설치한다.

<<<<<<< HEAD
#### \[트러블 슈팅]

설치 관련해서 오류가 많다.\
오류가 날경우,\

=======
### [트러블 슈팅]

설치 관련해서 오류가 많다.<br>
오류가 날경우,<br>
>>>>>>> parent of 4d4a984 (GITBOOK-5: change request with no subject merged in GitBook)

1. ./unstack.sh을 하여 서비스 관련 구성 및 네트워크를 정리한다.
2. ./clean.sh을 하여 openstack 관련 남은 파일을 모두 삭제한다.
3. sudo apt-get update로 패키지를 업데이트 해준다.

<<<<<<< HEAD
![img](../Img/openstack33.png)\
vi /etc/netplan의 yaml파일이 정확하게 입력되었는지 그리고\
local.conf의 PUBLIC\_NETWORK\_GATEWAY="192.168.0.1"이 제대로 입력되었는지 확인한다.\


\[설치중 git clone에서 멈추고 종료]\
local.conf에\


> GIT\_BASE=${GIT\_BASE:-https://git.openstack.org}

를 추가한다.\
하지만, 안될 수 있다.\
그래서 다시 unstack.sh를 하고 stack.sh으로 다시 설치하면서 될때까지 하면 되긴 한다.\


다른 오류관련해서는 구글링을 해야된다.\
(ChatGPT에 물어보면 도움이 된다.)\


![img](../Img/openstack34.png)\
설치완료 화면이다.\
설치가 완료되면 http://192.168.56.103/dashboard 와같이 openstack 대쉬보드 URL을 출력해준다.\


실제로 접속하면,\
![img](../Img/openstack35.png)\
접속이 되는것을 확인할 수 있다.\


#### \[중요]

현재 상태를 스냅샷으로 저장해두자.\


![img](../Img/openstack36.png)\
해당 가상 머신을 오른쪽 클릭한뒤 스냅샷을 클릭한다.

![img](../Img/openstack37.png)\
찍기를 누른뒤 다음과 같이 입력한뒤 확인을 누른다.\
=======
![img](../Img/openstack33.png)<br>
vi /etc/netplan의 yaml파일이 정확하게 입력되었는지 그리고<br>
local.conf의 PUBLIC_NETWORK_GATEWAY="192.168.0.1"이
제대로 입력되었는지 확인한다.<br>

[설치중 git clone에서 멈추고 종료] <br>
local.conf에<br>

> GIT_BASE=${GIT_BASE:-https://git.openstack.org}

를 추가한다.<br>
하지만, 안될 수 있다.<br>
그래서 다시 unstack.sh를 하고 stack.sh으로 다시 설치하면서 될때까지 하면 되긴 한다.<br>

다른 오류관련해서는 구글링을 해야된다.<br>
(ChatGPT에 물어보면 도움이 된다.)<br>

![img](../Img/openstack34.png)<br>
설치완료 화면이다.<br>
설치가 완료되면 http://192.168.56.103/dashboard 와같이
openstack 대쉬보드 URL을 출력해준다.<br>

실제로 접속하면,<br>
![img](../Img/openstack35.png)<br>
접속이 되는것을 확인할 수 있다.<br>

### [중요]

현재 상태를 스냅샷으로 저장해두자.<br>

![img](../Img/openstack36.png)<br>
해당 가상 머신을 오른쪽 클릭한뒤 스냅샷을 클릭한다.

![img](../Img/openstack37.png)<br>
찍기를 누른뒤 다음과 같이 입력한뒤 확인을 누른다.<br>
>>>>>>> parent of 4d4a984 (GITBOOK-5: change request with no subject merged in GitBook)

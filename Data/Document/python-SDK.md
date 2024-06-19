# Python SDK

Horizon 대시보드를 통해서 Openstack을 관리할 수 있지만 CLI를 통해 관리도 가능하다.<br>
참고 : 예시 환경은 kolla-ansible에서 MulitNode로 구축한 환경이다.
[MultiNode](/Data/Document/kolla-ansible_multinode.md)

## 설치 방법

> pip install python-openstackclient -c https://releases.openstack.org/constraints/upper/yoga


## 간단한 테스트

> kolla-ansible post-deploy

admin 관리자 자격 증명 설정 파일을 자동으로 생성해준다. <br>
admin.rc를 생성하자.

> source /etc/kolla/admin-openrc.sh

admin 인증 정보를 쉘에 로드한다.

> openstack service list

오픈스택 서비스의 리스트를 출력해본다.

![img](/Data/Img/python-sdk//3.png)

## Python 파일로 운영
#### [Python SDK](https://docs.openstack.org/newton/user-guide/sdk.html) (모든 예시는 공식 문서를 참고했다.)

### (0) 사전 작업
기본적으로 호스트 PC는 VM에 접근 가능하다. <br>
호스트 PC에서 openstack을 파이썬으로 관리해보자. <br>
참고 : 편집기는 VSCode이며 파이썬이 설치된것을 가정한다. <br>

![img](/Data/Img/python-sdk//5.png)

> pip install openstacksdk

openstacksdk를 설치한다.


### (1) 서비스 리스트 출력
openstack service list처럼 출력하는 예시를 작성해보자.

```
import openstack

conn = openstack.connect(
        auth_url="http://192.168.56.240:5000/v3",
        project_name="admin",
        username="admin",
        password="1q2w3e4r",
        user_domain_name="Default",
        project_domain_name="Default"
        )

strFormat = '%-33s%-2s%-12s%-2s%-10s\n'
strOut = strFormat % ('ID','|','Name','|','Type')
strOut += '-' * 65 + '\n'

services = conn.identity.services()
for service in services:
    strOut += strFormat %(service.id,'|',service.name,'|',service.type)

print(strOut)

# ex1.py
```
![img](/Data/Img/python-sdk//4.png)

### (2) 인스턴스, 이미지, flavor, 네트워크 정보 출력
```
import openstack

conn = openstack.connect(
        auth_url="http://192.168.56.240:5000/v3",
        project_name="admin",
        username="admin",
        password="1q2w3e4r",
        user_domain_name="Default",
        project_domain_name="Default"
        )

print("인스턴스")
print("=" * 50)
for server in conn.compute.servers():
    print(server)

print()

print("이미지")
print("=" * 50)

glance = conn.image
images = glance.images()

for image in images:
    print(f"ID: {image.id}")
    print(f"Name: {image.name}")
    print(f"Status: {image.status}")
    print(f"Size: {image.size / (1024 * 1024)} MB")
    print("-" * 50)

print()

print("Flavor")
print("=" * 50)

nova = conn.compute

flavors = nova.flavors()

for flavor in flavors:
    print(f"ID: {flavor.id}")
    print(f"Name: {flavor.name}")
    print(f"vCPUs: {flavor.vcpus}")
    print(f"RAM: {flavor.ram} MB")
    print(f"Disk: {flavor.disk} GB")
    print("-" * 50)

print()

print("네트워크")
print("=" * 50)

neutron = conn.network

networks = neutron.networks()

for network in networks:
    print(f"ID: {network.id}")
    print(f"Name: {network.name}")
    print(f"Status: {network.status}")
    
    print("-" * 50)

# ex2.py
```

![img](/Data/Img/python-sdk//6.png)<br>
다음과 같이 처음 구축하면 기본적인 이미지,flavor 네트워크가 구성되어 있지 않다.<br>
그래서, 오픈스택에서 제공하는 init_runonce를 사용해본다.<br>

#### [Controller]

> sudo vi ~/kolla/share/kolla-ansible/init-runonce

구성하기전에 네트워크 정보를 수정한다.

```
ENABLE_EXT_NET=${ENABLE_EXT_NET:-1}
EXT_NET_CIDR=${EXT_NET_CIDR:-'192.168.56.0/24'}
EXT_NET_RANGE=${EXT_NET_RANGE:-'start=192.168.56.100,end=192.168.56.110'}
EXT_NET_GATEWAY=${EXT_NET_GATEWAY:-'192.168.56.1'}

....

$KOLLA_OPENSTACK_COMMAND router create demo-router

$KOLLA_OPENSTACK_COMMAND network create demo-net
$KOLLA_OPENSTACK_COMMAND subnet create --subnet-range 10.0.10.0/24 --network demo-net \
    --gateway 10.0.10.1 --dns-nameserver 8.8.8.8 demo-subnet
$KOLLA_OPENSTACK_COMMAND router add subnet demo-router demo-subnet

# ~/kolla/share/kolla-ansible/init-runonce
```

> cd

> ./kolla/share/kolla-ansible/init-runonce

init-runonce를 실행하고 다시 출력해본다.

```
인스턴스
==================================================

이미지
==================================================
ID: 87a70e2a-7892-4a07-88e6-1af45f18d761
Name: cirros
Status: active
Size: 20.16943359375 MB
--------------------------------------------------

flavor
==================================================
ID: 1
Name: m1.tiny
vCPUs: 1
RAM: 512 MB
Disk: 1 GB
--------------------------------------------------
ID: 2
Name: m1.small
vCPUs: 1
RAM: 2048 MB
Disk: 20 GB
--------------------------------------------------
ID: 3
Name: m1.medium
vCPUs: 2
RAM: 4096 MB
Disk: 40 GB
--------------------------------------------------
ID: 4
Name: m1.large
vCPUs: 4
RAM: 8192 MB
Disk: 80 GB
--------------------------------------------------
ID: 5
Name: m1.xlarge
vCPUs: 8
RAM: 16384 MB
Disk: 160 GB
--------------------------------------------------
ID: 6
Name: m2.tiny
vCPUs: 2
RAM: 512 MB
Disk: 1 GB
--------------------------------------------------

네트워크
==================================================
ID: e07637df-a581-4cf1-a2c6-785236c33365
Name: demo-net
Status: ACTIVE
--------------------------------------------------
ID: f9001b1a-a84a-4710-a359-0b3ef1a45541
Name: public1
Status: ACTIVE
--------------------------------------------------
```

### (3) 인스턴스 생성
이미지, flavor, 네트워크 정보들을 활용해 인스턴스를 생성해본다.

```
import openstack

conn = openstack.connect(
        auth_url="http://192.168.56.240:5000/v3",
        project_name="admin",
        username="admin",
        password="1q2w3e4r",
        user_domain_name="Default",
        project_domain_name="Default"
        )

image = conn.compute.find_image("cirros")
flavor = conn.compute.find_flavor("m1.tiny")
network = conn.network.find_network("demo-net")

server = conn.compute.create_server(
    name="test",
    image_id=image.id,
    flavor_id=flavor.id,
    networks=[{"uuid": network.id}],
    key_name="mykey"
)

server = conn.compute.wait_for_server(server) 
# 인스턴스를 생성할때까지 기다린다.

print("인스턴스")
print("=" * 50)
for server in conn.compute.servers():
    print(server)

# ex3.py
```

![img](/Data/Img/python-sdk//7.png)<br>
생성된것을 확인할 수 있다.
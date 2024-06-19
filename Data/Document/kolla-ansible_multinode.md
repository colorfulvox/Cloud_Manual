# MultiNode

## Environment

- Hypervisor : virtuaBox
- OS : Ubuntu 20.04
- Controller : Controller에 네크워크, 스토리지 서비스가 설치된다.
    - vCPU : 4
    - Mem : 8192MB
    - Disk : /dev/sda(root)=100GB, /dev/sdb(cinder volumes)=100GB
    - Network Interface
        - NAT : 10.0.2.15 [패키지 설치용]
        - 호스트 전용 어댑터 : 192.168.56.240 [내부 네트워크]
        - 어댑터에 브리지 : No-IP [Neutron 인터페이스]
- Compute
    - vCPU : 4
    - Mem : 8192MB
    - Disk : /dev/sda(root)=100GB
    - Network Interface
        - NAT : 10.0.2.15
        - 호스트 전용 어댑터 : 192.168.56.241
        - 어댑터에 브리지 : No-IP
- Openstack Version : Yoga
> git+https://opendev.org/openstack/kolla-ansible@unmaintained/yoga

## START

### 0. 우분투 설치 생략

#### [Controller & Compute]

> sudo apt update


### 1. 네트워크 구성
내부 네트워크, neutron용 인터페이스를 수정한다.

#### [Controller]
> sudo vi /etc/netplan/00-installer-config.yaml

```
network:
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: false
      addresses: [192.168.56.240/24]
    enp0s9: {} # neutron용 인터페이스이기에 IP를 비워둔다.
  version: 2

# /etc/netplan/00-installer-config.yaml
```
> sudo netplan apply

#### [Compute]
> sudo vi /etc/netplan/00-installer-config.yaml

```
network:
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: false
      addresses: [192.168.56.241/24]
    enp0s9: {}
  version: 2

# /etc/netplan/00-installer-config.yaml
```
> sudo netplan apply


### 2. 유저 생성

Ansible의 SSH 접속을 위한 동일한 유저 계정 생성

[Controller & Compute]

> sudo su

> adduser stack

패스워드는 1q2w3e4r로 통일함
> usermod -aG sudo stack

> vi /etc/sudoers

```
stack ALL=(ALL) NOPASSWD:ALL

# /etc/sudoers
```
Asnible이 SSH접속하여 관리자 권한이 필요할때가 있다. <br>
패스워드없이 root 계정으로 전환할 수 있게 한다.

### 3. 시간 동기화
Node간에 시간 동기화가 필요하다.

#### [Controller]
> sudo apt-get install chrony

> sudo vi /etc/chrony/chrony.conf

```
# pool ntp.ubuntu.com        iburst maxsources 4
# pool 0.ubuntu.pool.ntp.org iburst maxsources 1
# pool 1.ubuntu.pool.ntp.org iburst maxsources 1
# pool 2.ubuntu.pool.ntp.org iburst maxsources 2
server time.bora.net iburst

allow 0/0

# /etc/chrony/chrony.conf
```
> sudo timedatectl set-timezone Asia/Seoul

> sudo systemctl restart chronyd

> sudo systemctl enable chronyd

#### [Compute]

> sudo apt-get install chrony

> sudo vi /etc/chrony/chrony.conf

```
# pool ntp.ubuntu.com        iburst maxsources 4
# pool 0.ubuntu.pool.ntp.org iburst maxsources 1
# pool 1.ubuntu.pool.ntp.org iburst maxsources 1
# pool 2.ubuntu.pool.ntp.org iburst maxsources 2
server 192.168.56.240 iburst

allow 0/0

# /etc/chrony/chrony.conf
```
> sudo timedatectl set-timezone Asia/Seoul

> sudo systemctl restart chronyd

> sudo systemctl enable chronyd


#### [시간 동기화 테스트]

> chronyc tracking

Controller와 Compute 모두 동일하게 Leap status : Normal이 뜨면 정상이다.

> chronyc sources

등록된 IP address 옆에 ^*이 떠있으면 정상이다.

> date

현재 한국 시간이면 정상이다.

> sudo chronyc clients

Controller에 Compute노드가 있으면 정상이다.

#### [오류 사항]

(1) 동기화가 안되는 경우

> sudo apt-get --purge remove chronyd

패키지를 삭제하고 다시 시도한다.
 
> sudo systemctl stop chronyd

> sudo systemctl start chronyd

패키지를 중지하고 다시 시작을 반복한다.

(2) Deploy 과정에서 시간 동기화가 끊기는 경우

```
prechecks_enable_host_ntp_checks: "false"

globals.yml
```

kolla-ansible의 globals.yml에서 ntp 체크를 해제한다.

#### [주의사항]

버츄얼박스에서 VM의 스냅샷을 기록하고 다시 실행하면 한국 시간이 맞춰지지 않는다. <br>
그래서 반드시 sudo systemctl enable chronyd 하거나 실행한뒤 date로 시간을 확인한다.


### 4. 디스크 볼륨 생성
Controller의 /dev/sdb 파티션을 LVM으로 관리하기 위해 설정한다.


#### [Controller]

> sudo pvcreate /dev/sdb

> sudo vgcreate cinder-volumes /dev/sdb


#### [그룹 체크]

> sudo vgs

cinder-volumes가 있으면 정상이다.

### 5. 패키지 설치

#### [Controller]

> su stack

> sudo apt install git python3-dev libffi-dev gcc libssl-dev

> sudo apt install python3-venv

> cd ~

> python3 -m venv ./kolla

> source ./kolla/bin/activate

(kolla)stack@hostname 이면 정상이다.

> pip install -U pip

> pip install 'ansible>=4,<6'

> pip install git+https://opendev.org/openstack/kolla-ansible@unmaintained/yoga


### 6. 환경 구성

#### [controller]

> sudo mkdir -p /etc/kolla

> sudo chown $USER:$USER /etc/kolla

> cp -r ./kolla/share/kolla-ansible/etc_examples/kolla/* /etc/kolla

> cp -r ./kolla/share/kolla-ansible/ansible/inventory/* /etc/kolla

/etc/kolla 위치에 [all-in-one, multinode, golbals.yml, passwords.yml]이 있으면 정상이다.

---
(1) Ansible의 환경 설정 파일을 구성한다.

> sudo mkdir -p /etc/ansible

> sudo vi /etc/ansible/ansible.cfg

```
[defaults]
host_key_checking=False
pipelining=True
forks=100

# /etc/ansible/ansible.cfg
```
---
(2) MultiNode 인벤토리 파일을 수정한다.

> vi /etc/kolla/multinode

```
# [all:vars]
# ansible_become_pass='1q2w3e4r'
# 만약 root 계정 접속이 제한되면 추가한다.

[control]
control01       ansible_user=stack ansible_host=192.168.56.240 ansible_become=true

[network]
network01       ansible_user=stack ansible_host=192.168.56.240 ansible_become=true

[compute]
compute01       ansible_user=stack ansible_host=192.168.56.241 ansible_become=true

[monitoring]

[storage]
storage01       ansible_user=stack ansible_host=192.168.56.240 ansible_become=true

[deployment]
localhost       ansible_connection=local ansible_become=true

...

# /etc/kolla/multinode
```
---
(3) Ansible이 SSH 접속할 수 있도록 SSH키를 생성한뒤 pub키를 전송한다.

> ssh-keygen

> ssh-copy-id stack@192.168.56.240

> ssh-copy-id stack@192.168.56.241

---
(4) MultiNode 인벤토리에 구성한 노드들의 통신을 체크한다.

> cd /etc/kolla

> ansible -i multinode all -m ping

모든 그룹의 Ping 테스트가 Success면 정상이다.

---
Openstack 서비스들의 비밀번호를 생성한다. <br>
참고 : Keystone Admin 비밀번호를 수정한다.

> kolla-genpwd

> vi passwords.yml

```
...
keystone_admin_password: 1q2w3e4r
...

# passwords.yml
```

---
(5) Kolla-ansible의 전역 설정 파일을 수정한다.

> vi globals.yml

```
kolla_base_distro: "ubuntu"
kolla_install_type: "source"
openstack_release: "yoga"
kolla_internal_vip_address: "192.168.56.245"
network_interface: "enp0s8"
neutron_external_interface: "enp0s9"
neutron_plugin_agent: "openvswitch"
enable_openstack_core: "yes"
enable_haproxy: "yes"
enable_neutron_provider_networks: "yes"
nova_compute_virt_type: "kvm"
enable_cinder: "yes"
enable_cinder_backend_lvm: "yes"

# globals.yml
```

### 7. 배포

배포하기전 반드시 VM의 스냅샷을 기록해두자. <br>
(오류가 많이 발생한다...)

> kolla-ansible install-deps

Kolla-ansible의 의존성 패키지와 도구를 설치한다.

> kolla-ansible -i ./multinode bootstrap-servers

초기 설정을 진행한다.

#### [오류]

    E: Could not get lock /var/lib/dpkg/lock-frontend - open (11: Resource temporarily unavailable)
    E: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), is another process using it?

    해결 방법)

    #### [Controller & Compute]

    > sudo killall apt apt-get

    > sudo rm /var/lib/apt/lists/lock

    > sudo rm /var/cache/apt/archives/lock

    > sudo rm /var/lib/dpkg/lock*

    > sudo rm -r /var/lib/apt/lists/* -vf

    > sudo dpkg --configure -a

    > sudo apt clean

    > sudo apt-get update


    이래도 해결이 안된다면 스냅샷을 복구해 다시 시도하거나,
    bootstrap-servers 명령어를 반복해서 수행한다.


> kolla-ansible -i ./multinode prechecks

설정이 제대로 되었는지 체크한다.

#### [오류]

    fatal: [compute01]: FAILED! => {"changed": false, "msg": "timedatectl sees the system clock as unsynchronized. Please wait for synchronization. Alternatively, set 'prechecks_enable_host_ntp_checks' to 'false' to disable this check if your NTP daemon is not recognised by 'timedatectl status'."

    해결 방법)

    시간 동기화를 했는데 동기화가 끊어지는 현상이 있다.
    globals.yml에서 prechecks_enable_host_ntp_checks: "false" 한다.


> kolla-ansible -i ./multinode pull

MultiNode 인벤토리에 필요한 도커 이미지를 가져온다.

> kolla-ansible -i ./multinode deploy

##### 배포를 진행한다. 기도하자.

배포를 실패하면 오류 사항을 구글링해서 수정한뒤)
> kolla-ansible destroy -i ./multinode --include-images --yes-i-really-really-mean-it

> kolla-ansible prune-images -i ./multinode --yes-i-really-really-mean-it

배포를 해제하고 다시 진행한다.

### 8. 결과

[Controller & Compute]

> sudo docker ps

![img](/Data/Img/kolla-ansible/1.png)

컨테이너로 서비스들이 구성되었는지 확인하다.

![img](/Data/Img/kolla-ansible/2.png)

> http://192.168.56.245/

대시보드로 접속이 되는지 확인하고 admin으로 로그인도 해본다.

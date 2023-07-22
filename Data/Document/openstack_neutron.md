[뒤로가기](../../README.md)<br>

# 네트워크 서비스 Neutron

### [Nova-Network => Neutron](https://blog.naver.com/love_tolty/220237750951)

Neutron에 대한 구체적인 설명을 해주는 블로그 링크이다.<br>

과거 Nova-Network는 구성된 모든 Compute 노드들이<br>
DB에 직접 접근에 대한 권한과 그에 따른 부하가 있었다.<br>
또한, 외부 통신을 할때 Floating IP 범위의 적어 IP 대역이
한정적이였다.<br>

결론적으로 Nova-Network는 대규모 클라우드 환경 구성에 어려웠다.<br>

그래서 이런 제한적인 IP 대역 문제를 해결하기위해 SDN이 등장했으며,<br> SDN을 이용해 인스턴스의 Floating/Fixed IP를 관리하는 Neutron이 등장했다.<br>

Neutron은 인스턴스의 가상 네트워크를 구성하고 관리한다. <br>리소스에는 라우터, 서브넷, 포트, 네트워크가 포함되며<br>
해당 리소스를 활용해 VM과 연결하고 트래픽을 전달하며 방화벽 규칙을 적용할 수 있다.<br>

### [참고]

Neutron은 네트워크의 생성/변경/삭제 대한 API만 제공할뿐<br>
실제로는 플러그인을 통해 네트워크가 구성된다.<br>
그래서 서비스 환경에 따라 필요한 플러그인을 적용해 사용하면 된다.<br>

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

## [Controller]

### (1) DB 생성 및 Keystone 계정 생성 [Controller]

다른 서비스와 똑같이 DB를 생성하고<br>
Keystone의 사용자, 서비스, 엔드포인트를 설정한다. <br>

![img](../Img/openstack_202.png)

```
mysql -u root -popenstack

CREATE DATABASE neutron;

GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'neutrondbpass';

GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'neutrondbpass';

exit;
```

DB 생성, 비밀번호는 neutrondbpass로 설정한다.<br>

![img](../Img/openstack_203.png)<br>

> . adminrc.sh<br>
> openstack user create --domain default --password-prompt neutron

관리자로 접속한 뒤, neutron 계정을 생성한다.<br>
비밀번호는 neutronpass로 설정한다.<br>

![img](../Img/openstack_204.png)<br>

> openstack role add --project service --user neutron admin<br>
> openstack service create --name neutron --description "OpenStack Networking" network<br>
> openstack endpoint create --region RegionOne network public http://controller:9696<br>
> openstack endpoint create --region RegionOne network internal http://controller:9696<br>
> openstack endpoint create --region RegionOne network admin http://controller:9696<br>

관리자 권한을 부여하고, 서비스 생성 및 엔드 포인트를 설정한다.<br>

### (2) neutron 설치 및 환경 설정 [Controller]

neutron 설치에는 두가지 방식이 있다.<br>

1\_ 공급자 네트워크<br>
외부 네트워크에 대한 인스턴스 연결만 지원하는 가장 단순한 아키텍쳐이다.<br>
셀프(사설) 네트워크, 라우터와 유동 IP주소가 없다.<br>
또한, 권한만 있는 사용자만 admin 공급자 네트워크를 관리 할 수 있다.<br>

2\_ 셀프 서비스 네트워크<br>
인스턴스 연결을 지원하며 권한이 없는 다른 demo사용자도 라우터를 포함하여 네트워크를 관리할 수 있다.<br>
또한, 유동 IP주소는 외부 네트워크에서 셀프 서비스 네트워크를 사용하여 인스턴스 연결을 제공한다.<br>

여기서는 셀프 서비스 네트워크 방식으로 구성한다.<br>

![img](../Img/openstack_205.png)<br>

```
apt-get install neutron-server neutron-plugin-ml2 \
  neutron-linuxbridge-agent neutron-l3-agent neutron-dhcp-agent \
  neutron-metadata-agent
```

neutron-server : neutron 서비스 패키지이다.
네트워크 리소스를 관리하고 API 요청을 처리해 네트워크 구성 정보를 저장하고 다른 Neutron-agent와 상호 작용한다.

neutron-plugin-ml2 : neutron 플러그인이다.<br>
ML2는 다양한 네트워크 기술과 드라이버를 통합하여 가상 네트워크를 구성하고 관리할 수 있도록 제공한다.<br>
VLAN, VXLAN, GRE등의 프로토콜들을 사용하여 가상 네트워크를 구성할 수 있다.<br>

neutron-linuxbridge-agent : 가상 머신과 호스트의 네트워크를
연결한다.<br>
linuxbridge를 사용하여 가상 네트워크를 생성해 데이터 패킷을 전달한다.<br>

neutron-l3-agent : 가상 라우터를 생성해 가상 네트워크간의 패킷 라우팅을 처리한다.<br>

neutron-dhcp-agent : 가상 머신에 대한 IP주소와 네트워크 설정을
동적으로 할당한다.<br>
우분투를 처음설치할때 네트워크 설정이 dhcp로 설정되어 있는 것처럼<br>
가상 머신이 네트워크로 자동으로 연결이 되도록 한다.<br>

neutron-metadata-agent : 가상 머신의 인스턴스에 대한 메타데이터 정보를 제공한다.<br>

[neutron.conf 설정]

![img](../Img/openstack_206.png)<br>

> vi /etc/neutron/neutron.conf

neutron 설정 파일에 들어간다.

![img](../Img/openstack_207.png)<br>

```
service_plugins = router
allow_overlapping_ips = true
transport_url = rabbit://openstack:rabbitpass@controller
auth_strategy = keystone

notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true
```

service_plugins = router<br>
Neutron의 서비스 플러그인을 지정한다.<br>
여기서는 L3 네트워킹 기능을 사용하는 라우터 플러그인을 사용한다.<br>

allow_overlapping_ips = true<br>
중복 IP 주소를 허용한다.<br>
여러 가상 네트워크에서 같은 IP 주소를 사용할 수 있다.<br>

notify_nova_on_port_status_changes = true<br>
Neutron의 포트 상태 변경시 Nova에게 알려준다.<br>

notify_nova_on_port_data_changes = true<br>
Neutron의 포트 데이터 변경시 Nova에게 알려준다.<br>

![img](../Img/openstack_208.png)<br>

> connection = mysql+pymysql://neutron:neutrondbpass@controller/neutron

![img](../Img/openstack_209.png)<br>

```
www_authenticate_uri = http://192.168.56.101:5000
auth_url = http://192.168.56.101:5000
memcached_servers = 192.168.56.101:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = neutronpass
```

![img](../Img/openstack_210.png)<br>

```
auth_url = http://192.168.56.101:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = novapass
```

![img](../Img/openstack_211.png)<br>

> lock_path = /var/lib/neutron/tmp

---

[ML2 플러그인 설정]

![img](../Img/openstack_212.png)<br>

> vi /etc/neutron/plugins/ml2/ml2_conf.ini

ml2 설정 파일에 들어간다.

![img](../Img/openstack_213.png)<br>

```
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
mechanism_drivers = linuxbridge,l2population
extension_drivers = port_security
```

type_drivers = flat,vlan,vxlan<br>
드라이브를 지정한다.<br>
flat : 가장 기본적인 Nova-Network의 구성 모델로<br>
인스턴스는 IP pool에서 Fixed IP를 할당받는다.<br>

vlan : 테넌트별 트래픽을 분리하기 위해 개선된 모델이다.<br>

vxlan : vlan의 확장된 모델로 더많은 네트워크를 구성할 수 있다.<br>

tenant_network_types = vxlan<br>
테넌트는 클라우드 환경에서 개별적으로 관리되는 가상의 공간이다.<br> 일반적으로 조직,프로젝트,부서와 관련된 가상의 컴퓨팅 리소스의 집합을 가리킨다.<br>

테넌트 네트워크는 클라우드 환경에서 특정 테넌트에 의해 사용되는 가상 네트워크이다.<br>

다중 테넌트 환경에서 각 테넌트에는 자체 논리 네트워크가 필요하며,<br>
이에 따라 자체 네트워크 식별이 필요하다.<br>
과거 VLAN을 사용하여 테넌트를 분리했지만, 한정적인 주소 공간으로<br>
큰 규모의 네트워크 구성이 어려웠다.<br>
그래서 VXLAN을 통해 주소 공간을 확장했다.<br>

![img](../Img/openstack_214.png)<br>

> flat_networks = provider

![img](../Img/openstack_215.png)<br>

> vni_ranges = 1:1000

![img](../Img/openstack_216.png)<br>

> enable_ipset = true

---

[linux_bridge 설정]

![img](../Img/openstack_217.png)<br>

> vi /etc/neutron/plugins/ml2/linuxbridge_agent.ini

linux_bridge 설정 파일에 들어간다.

![img](../Img/openstack_218.png)<br>

> physical_interface_mappings = provider:enp0s8

![img](../Img/openstack_219.png)<br>

> firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver<br>
> enable_security_group = true

![img](../Img/openstack_220.png)<br>

> enable_vxlan = true<br>
> local_ip = 192.168.56.101<br>
> l2_population = true<br>

[linux bridge 활성화]

![img](../Img/openstack_248.png)<br>

> vi /etc/sysctl.conf

![img](../Img/openstack_249.png)<br>

> net.bridge.bridge-nf-call-iptables = 1<br>
> net.bridge.bridge-nf-call-ip6tables = 1

![img](../Img/openstack_250.png)<br>

> sysctl -p

[l3 설정]

![img](../Img/openstack_221.png)<br>

> vi /etc/neutron/l3_agent.ini

l3 설정 파일에 들어간다.

![img](../Img/openstack_222.png)<br>

> interface_driver = linuxbridge

[DHCP 설정]

![img](../Img/openstack_223.png)<br>

> vi /etc/neutron/dhcp_agent.ini

dhcp 설정 파일에 들어간다.

![img](../Img/openstack_224.png)<br>

```
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = true
```

[metadata 설정]

![img](../Img/openstack_225.png)<br>

> vi /etc/neutron/metadata_agent.ini

메타데이터 설정 파일에 들어간다.

![img](../Img/openstack_226.png)<br>

> nova_metadata_host = controller<br>
> metadata_proxy_shared_secret = metasecret001

[nova 설정]

![img](../Img/openstack_227.png)<br>

> vi /etc/nova/nova.conf

![img](../Img/openstack_228.png)<br>

```
auth_url = http://192.168.56.101:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = neutronpass
service_metadata_proxy = True
metadata_proxy_shared_secret = metasecret001
```

### (3) DB 연동 [Controller]

![img](../Img/openstack_229.png)<br>

```
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
--config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
```

### (4) 실행 테스트 [Controller]

![img](../Img/openstack_230.png)<br>

> service nova-api restart<br>
> service nova-api status

![img](../Img/openstack_231.png)<br>

```
service neutron-server restart
service neutron-linuxbridge-agent restart
service neutron-dhcp-agent restart
service neutron-metadata-agent restart
service neutron-l3-agent restart
```

![img](../Img/openstack_232.png)<br>

```
service neutron-server status
service neutron-linuxbridge-agent status
service neutron-dhcp-agent status
service neutron-metadata-agent status
service neutron-l3-agent status
```

## [Compute]

Compute 노드는 인스턴스에 대한 연결 및 보안 그룹 구성을
해야한다.<br>
그래서 linuxbridge를 설치한다.<br>

### (5) linuxbridge 설치 및 설정 [Compute]

![img](../Img/openstack_233.png)<br>

> apt-get install neutron-linuxbridge-agent

linuxbridge를 설치한다.

[neutron 설정]

![img](../Img/openstack_234.png)<br>

> vi /etc/neutron/neutron.conf

설정 파일에 들어간다.

![img](../Img/openstack_235.png)<br>

> transport_url = rabbit://openstack:rabbitpass@192.168.56.101<br>
> auth_strategy = keystone

core_plugin을 주석처리한다.

![img](../Img/openstack_236.png)<br>

connection을 주석 처리한다.

![img](../Img/openstack_237.png)<br>

```
www_authenticate_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = neutronpass
```

![img](../Img/openstack_238.png)<br>

> lock_path = /var/lib/neutron/tmp

[linuxbridge 설정]

![img](../Img/openstack_239.png)<br>

> vi /etc/neutron/plugins/ml2/linuxbridge_agent.ini

linuxbridge 설정 파일에 들어간다.

![img](../Img/openstack_240.png)<br>

> physical_interface_mappings = provider:enp0s8

![img](../Img/openstack_242.png)<br>

> firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver<br>
> enable_security_group = true

![img](../Img/openstack_241.png)<br>

```
enable_vxlan = true
local_ip = 192.168.56.102
l2_population = true
```

[linux bridge 활성화]

![img](../Img/openstack_251.png)<br>

> vi /etc/sysctl.conf

![img](../Img/openstack_253.png)<br>

> net.bridge.bridge-nf-call-iptables = 1<br>
> net.bridge.bridge-nf-call-ip6tables = 1

![img](../Img/openstack_252.png)<br>

> sysctl -p

[nova 설정]

![img](../Img/openstack_243.png)<br>

> vi /etc/nova/nova.conf

![img](../Img/openstack_244.png)<br>

```
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = neutronpass
```

### (6) 실행 테스트 [Compute]

![img](../Img/openstack_245.png)<br>

> service nova-compute restart<br>
> service nova-compute status<br>
> service neutron-linuxbridge-agent restart<br>
> service neutron-linuxbridge-agent status<br>

### (7) 최종 테스트 [Compute]

![img](../Img/openstack_246.png)<br>

> . adminrc.sh
> neutron ext-list

![img](../Img/openstack_247.png)<br>

> neutron agent-list

## [참고]

### 각 서비스를 설치한뒤 반드시 VM의 스냅샷을 찍어 기록해두자.

Openstack 설치 과정에서 에러가 날 경우 다시 복원할 수 있어야한다.<br>

> 네트워크 구성 내용이 많아 따로 정리해야겠다.

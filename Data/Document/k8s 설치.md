[뒤로가기](../../README.md)<br>

# 쿠버네티스 설치

master, node1, node2 3개의 가상 머신을 구성한뒤
쿠버네티스를 설치하여 master에 노드들을 Join한다.<br>

## 환경

VM 가상 머신<br>
ubuntu - 20.04<br>

### (1) Master, Node1, Node2 네트워크 설정

![img](../Img/k8s_1.png)<br>

![img](../Img/k8s_2.png)<br>
만들기를 클릭한뒤 새로운 네트워크의 Name과
IPv4 Prefix : 10.100.0.0/24를 입력한다.<br>

![img](../Img/k8s_3.png)<br>
포트 포워딩을 클릭한뒤,<br>
node1 | TCP | 127.0.0.1 | 101 | 10.100.0.101 | 22<br>
node2 | TCP | 127.0.0.1 | 102 | 10.100.0.102 | 22<br>
master | TCP | 127.0.0.1 | 104 | 10.100.0.104 | 22<br>
를 입력해 적용한다.<br>

### (2) Ubuntu 가상 서버 생성

[VM 환경]<br>
name : k8s-master<br>
cpu : 2 core<br>
memory : 4096MB -> k8s를 설치한뒤 2GIB로 줄일것이다.<br>
network : nat network<br>
disk size : 20GIB<br>

[Ubuntu 서버 만들기](../Document/VM.md) -> 여기서는 [Ubntu 20.04](https://releases.ubuntu.com/focal/) 버전을 사용한다.<br>

MASTER VM을 만들어 환경 설정을 한뒤 node1, node2도 복제해 만들것이다.<br>

![img](../Img/k8s_4.png)<br>
네트워크는 생성한 VM의 설정에서 NAT 네트워크를 선택한뒤
방금 생성한 네트워크로 설정한다.<br>

### (3) Ubuntu 환경 설정

![img](../Img/k8s_5.png)<br>
가상 머신을 실행한다.<br>

![img](../Img/k8s_6.png)<br>
처음 환경 설정은 생략 -> 계속하기 클릭하면서 설치 한다.<br>

![img](../Img/k8s_7.png)<br>
설치가 완료되면 설정-> 네트워크로 들어간다.<br>

![img](../Img/k8s_8.png)<br>
유선의 톱니바퀴를 클릭해 설정에 들어간다.<br>

![img](../Img/k8s_9.png)<br>
IPv4탭에서 방식을 수동으로 설정<br>
주소<br>
10.100.0.104 | 24 | 10.100.0.1<br>
DNS<br>
10.100.0.1<br>
로 설정한다. (master 기준)<br>

![img](../Img/k8s_10.png)<br>
바탕화면에서 터미널을 연뒤

> sudo vi /etc/hostname

명령어로 호스트 네임을 변경한다.<br>

![img](../Img/k8s_11.png)<br>
기존 내용을 지운뒤,

> master.example.com

으로 입력한뒤 저장한다.<br>

![img](../Img/k8s_12.png)<br>

> sudo vi /etc/hosts

을 입력한뒤 현재 호스트네임과 다른 호스트도 등록 해준다.<br>

![img](../Img/k8s_13.png)<br>
다음과 같이 입력하고 저장한다.<br>

### (4) 패키지 설치

필요한 패키지를 설치하기전,
![img](../Img/k8s_14.png)<br>
소프트웨어 업데이트가 모두 끝난뒤 설치한다.<br>

![img](../Img/k8s_15.png)<br>
그리고 터미널을 열어

> sudo apt-get install -y openssh-server curl vim tree

입력해 설치한다.<br>

### (5) 터미널 연결

![img](../Img/k8s_16.png)<br>

> ssh master@127.0.0.1 -p 104

을 입력해 master vm에 접속한다.<br>

### (6) 도커 설치 - 생략

[Docker 설치](../Document/Docker.md)

### (7) 가상 머신 복제

![img](../Img/k8s_17.png)<br>
master 가상 머신을 오른쪽 클릭해 복제를 클릭한다.<br>

![img](../Img/k8s_18.png)<br>
k8s-node1으로 입력하고,
MAC Address Policy는
모든 네트워크 어댑터의 새 MAC 주소 생성을 선택<br>

그 다음 완전 복제를 선택한뒤 Finish한다.<br>

![img](../Img/k8s_19.png)<br>
복제가 완료되면 생성된 우분투를 시작해준다.<br>

![img](../Img/k8s_20.png)<br>

> sudo vi /etc/hostname

으로 들어가 node1.example.com 으로 변경한다.

![img](../Img/k8s_21.png)<br>
네트워크 주소도 바꿔준다.<br>

[참고] - 우분투 계정이 헷갈릴 수 있어
우분투 계정명을 master에서 chu로 변경했다.<br>
![img](../Img/k8s_22.png)<br>

![img](../Img/k8s_23.png)<br>
그리고 터미널로 접속이 가능한지 확인한다.<br>

![img](../Img/k8s_24.png)<br>
node2도 똑같이 복제해 생성한다.<br>
그리고 node1,node2의 메모리 크기를 2048MB로 변경한다.<br>

![img](../Img/k8s_25.png)<br>
최종적으로 master,node1,node2모두 접속이 가능한지 확인한다.<br>

### (8) 가상 머신 snapshot 저장

![img](../Img/k8s_26.png)<br>
모든 가상 머신을 끈뒤 master부터 스냅샷을 클릭한뒤
찍기를 누른다.<br>

![img](../Img/k8s_27.png)<br>
현재 docker만 설치된 상태이기에 설명에 다음과 같이 작성한뒤 확인을 눌러 저장한다.<br>

node1, node2도 똑같이 스냅샷을 만들어 저장한다.<br>

### (9) 쿠버네티스 설치

![img](../Img/k8s_28.png)<br>
다시 모든 가상머신에 접속한다.<br>

### [쿠버네티스 공식 문서](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

![img](../Img/k8s_29.png)<br>
공식 문서에 보면 다음과 같이 swap을 비활성화 해야된다.<br>

![img](../Img/k8s_30.png)<br>

> sudo -i

master를 관리자 계정으로 변경한다.<br>

![img](../Img/k8s_31.png)<br>

> swapoff -a && sed -i '/swap/s/^/#/' /etc/fstab

swap을 영구 비활성화 한다.

![img](../Img/k8s_32.png)<br>

br_netfilter는 방화벽인 iptables가 bridge driver를 사용하여 통신을 하는 패킷을 filter 할수 있도록 해준다.<br>
해당 기능을 활성화해야 Docker가 default로 사용하는<br> bridge driver를 사용하여 생성되는 container들간의 통신이 가능해진다.<br>
쿠버네티스에서는 이런 container를 포함하는게 Pod라한다.<br>
결론적으로 Pod끼리의 통신을 가능하게 하기 위한 설정이다.<br>

> lsmod | grep br_netfilter<br>
> modinfo br_netfilter<br>
> sudo modprobe br_netfilter<br>

하지만, 현재 확인한것과 같이 br_netfilter 커널이 존재하지만
사용되지 않고 있다.<br>
그리고 bridge 모듈에 의존해서 동작한다고 나온다.<br>

그래서, br_netfilter 모듈을 iptables가 사용할 수 있도록 설정하자.<br>

![img](../Img/k8s_33.png)<br>

> cat \<\<EOF > /etc/sysctl.d/k8s.conf <br>
> net.bridge.bridge-nf-call-ip6tables = 1 <br>
> net.bridge.bridge-nf-call-iptables = 1 <br>
> EOF<br>

> sysctl --system<br>

![img](../Img/k8s_34.png)<br>

> systemctl stop firewalld<br>
> systemctl disable firewalld<br>

firewall 방화벽이 존재하면 비활성화 한다.<br>

[주의사항]<br>
이제 k8s를 설치할텐데 현재 공식 문서에서
ubuntu 22.04 이전 버전에서는 /etc/apt/keyrings가 없다고 한다.<br>
그래서 설치하기전에<br>

> sudo mkdir -p /etc/apt/keyrings

해당 명령어로 keyrings폴더를 생성해둔다.<br>

> sudo apt-get update<br>
> sudo apt-get install -y apt-transport-https ca-certificates curl<br>

패키지를 업데이트 한뒤 k8s apt 레파지토리를 사용하는데 필요한 패키지 설치<br>

> curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg<br>

Google Cloud 공개 서명 키 설치<br>

> echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list<br>

k8s apt 레파지토리 추가<br>

> sudo apt-get update<br>
> sudo apt-get install -y kubelet kubeadm kubectl<br>

> sudo apt-mark hold kubelet kubeadm kubectl<br>

k8s를 설치하고 해당 버전을 고정<br>

![img](../Img/k8s_35.png)<br>

> sudo systemctl daemon-reload<br>
> sudo systemctl restart kubelet

시스템을 재구성하고 kubelet을 재시작

node1, node2에도 이 과정을 모두 수행한다.

### (10) 쿠버네티스 구성 - master에서만 실행

### 컴포넌트 초기화

![img](../Img/k8s_36.png)<br>

> kubeadm init

으로 초기화를 진행한다.<br>

![img](../Img/k8s_38.png)<br>
여기서 다음과 같은 에러가 뜰 경우<br>

> sudo rm /etc/containerd/config.toml<br>
> sudo systemctl restart containerd<br>

해당 명령어를 입력한다. <br>[containerd설정 파일인 config.toml 파일을 삭제]<br>

![img](../Img/k8s_37.png)<br>
해당 결과가 출력되면 성공이다.<br>

![img](../Img/k8s_39.png)<br>
다음과 같이 추후에 워커노드들을 연결하기 위해 토큰을 홈폴더에 저장한다.<br>

![img](../Img/k8s_40.png)<br>

> mkdir -p \$HOME/.kube<br>
> sudo cp -i /etc/kubernetes/admin.conf \$HOME/.kube/config <br>
> sudo chown \$(id -u):$(id -g) $HOME/.kube/config<br>
> cat > token.txt <br>
> 결과로 나온 token 정보

### Weave-Net 설치

> kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

![img](../Img/k8s_41.png)<br>
설치되었는지 확인하다.<br>

> kubectl get pod --all-namespaces<br>
> kubectl get nodes<br>

또한, master 노드가 준비중인지 확인<br>

### (11) worker node를 master에 join

![img](../Img/k8s_42.png)<br>

> cat token.txt<br>

master에 저장한 토큰 정보를 복사한다.<br>

![img](../Img/k8s_43.png)<br>
그리고 node1에 해당 정보를 붙여넣어 실행한다.<br>

여기서도,<br>
![img](../Img/k8s_38.png)<br>
와 같은 버그가 발생하면<br>

> sudo rm /etc/containerd/config.toml<br>
> sudo systemctl restart containerd<br>

> 해당 명령어를 입력하고 수행한다.<br>

node2도 똑같이 수행한다.<br>

![img](../Img/k8s_44.png)<br>
정상적으로 join했는지 확인하고<br>
master 노드에서 노드리스트를 출력해 등록이 되었는지 확인하다.<br>
또한, 기다리면 모두 Ready가 되는지 확인한다.<br>

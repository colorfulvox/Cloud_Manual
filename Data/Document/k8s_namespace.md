[뒤로가기](../../README.md)<br>

# k8s namespace

클러스터 하나를 여러개의 논리적인 단위로 나눠서 사용<br>
workernode에 여러 pod들을 실행해서 관리하기 복잡하지만,<br> namespace단위로 각 서비스별로 분리해 관리가 쉬워진다.<br>

k8s 클러스터 하나를 여러 팀이나 사용자가 함께 공유함<br>

### [CLI 명령어]

> kubectl get namepsaces

![img](../Img/k8s_namespace1.png)<br>

k8s가 운영되기 위해 기본적으로 만들어진 namespace이다.<br>

![img](../Img/k8s_namespace2.png)<br>

실제로 pod를 검색하면 default에 pod가 없다고 나온다.<br>
즉, namespace가 없이 pod를 생성하면 기본적으로 default그룹에 들어간다.<br>

![img](../Img/k8s_namespace3.png)<br>

실제 kube-system 네임스페이스에서 관리하는 pod를 보면 master node를 구성하기위한 pod들을 확인할 수 있다.<br>

![img](../Img/k8s_namespace4.png)<br>

> kubectl create namespace 그룹명<br>

### YAML 파일일 경우

kubectl create namespace 그룹명 --dry-run -o yaml

![img](../Img/k8s_namespace5.png)<br>

> kubectl create namespace orange --dry-run :<br>
> orange라는 이름의 namespace를 만들 수 있는지 확인하는 요청<br>
> -o yaml : 부가적으로 yaml파일 형태로 보여달라는 요청 <br> \> orange.yml : 만들어지면 orange.yml파일로 생성해달라는 요청<br>

![img](../Img/k8s_namespace6.png)<br>

> kubectl create -f orange.yml

### pod를 원하는 namespace에서 실행

### YAML 파일일 경우

![img](../Img/k8s_namespace7.png)<br>

> kubectl create -f nginx.yaml -n blue

아니면 pod의 yaml파일에서 설정해도 된다.<br>

![img](../Img/k8s_namespace8.png)<br>
yaml파일에 namespace를 지정한다.<br>

![img](../Img/k8s_namespace9.png)<br>
실제로 실행되는것을 볼 수 있다.<br>

### Base namespace를 변경

기본적으로 default namespace를 기준으로 생성되는것을<br>
원하는 namespace로 변경한다.<br>

그러기 위해서는 context등록이 필요하다.
<br>

![img](../Img/k8s_namespace10.png)<br>

> kubectl config view

현재는 admin에대한 context만 존재한다.

![img](../Img/k8s_namespace11.png)<br>

> kubectl config set-context blue@kubernetes --cluster=kubernetes --user=kubernetes-admin --namespace=blue

blue@kubernetes이름의 새로운 context를 만드는데
cluster는 kubernetes이고<br> 사용자는 admin
사용하는 namespace는 blue로 설정한다.<br>

![img](../Img/k8s_namespace12.png)<br>
실제로 blue namespace를 기본으로 사용하는 context가 생성되었다.
하지만,

> kubectl config current-context

현재 사용중인 context는 admin이기에 변경이 필요하다.

![img](../Img/k8s_namespace13.png)<br>

> kubectl config use-context blue@kubernetes

blue context를 사용하는것으로 변경한다.

![img](../Img/k8s_namespace14.png)<br>

실제로 blue namespace의 pod정보가 출력된다.
<br>

### namespace 삭제

namespace를 삭제하면 namespace에 속한 모든 pod들이 삭제된다.

![img](../Img/k8s_namespace15.png)<br>

> kubectl delete namespace blue

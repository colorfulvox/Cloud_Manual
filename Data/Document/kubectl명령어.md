[뒤로가기](../../README.md)

# kubectl 명령어

k8s 클러스터를 제어하기위한 커맨드 라인 도구

## 명령어 기본 구조

### kubectl [command] [type] [name] [flags]

command : obj에 실행할 명령어 (create, get, delete, edit....)<br>
type : obj 종류 (node, pod, service....)<br>
name : obj 이름 (웹서버같은 이름)<br>
flags : 부가적인 옵션 (-o options)<br>
예시) kubectl get pod webserver -o wide<br>
webserver의 pod정보를 자세하고 보여주는 요청<br>

## kubectl 명령어 자동 완성

kubectl은 명령어에 대한 자동 완성 기능을 제공하지 않는다.<br>
자동 완성 기능을 추가해주자.<br>

### [kubectl cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

![img](../Img/k8s_47.png)<br>
해당 부분들을 복사한다.<br>

> source <(kubectl completion bash)<br>
> echo "source <(kubectl completion bash)" >> ~/.bashrc<br>

![img](../Img/k8s_48.png)<br>
master node에 입력한다.<br>

![img](../Img/k8s_49.png)<br>
하지만, tab으로 자동 완성을 하면 에러가 난다.<br>

![img](../Img/k8s_50.png)<br>

> type \_init_completion

으로 자동 완성 기능이 활성화되어 있는지 확인한다.
안되어있다.

![img](../Img/k8s_51.png)<br>

> vi ~/.bashrc

bash 설정 파일에 들어간다.

![img](../Img/k8s_52.png)<br>
해당 부분의 주석을 모두 풀고 저장한다.<br>

![img](../Img/k8s_53.png)<br>

> exit<br>
> sudo su<br>
> kubectl api-re \<tab\><br>

관리자 계정으로 재로그인 하면 잘 된다.<br>

### [명령어 약어 정보]

> kubectl api-resources

### [노드 정보 보기]

> kubectl get nodes

> kubectl get nodes -o wide

> kubectl describe node master.example.com

### [pod 정보 보기]

> kubectl get pods

> kubectl describe pods webserver

### [컨테이너 pod 만들기]

> kubectl run webserver --image=nginx:1.14 --port 80

run : 1개의 컨테이너 pod를 실행하거나 만드는 명령어<br>
nginx 이미지를 쓰고 80 port를 사용하는 webserver 컨테이너 생성<br>

![img](../Img/k8s_54.png)<br>
실행중이다.<br>

![img](../Img/k8s_55.png)<br>

> apt-get install elinks

elinks : 터미널에서 웹 브라우저를 볼 수 있는 패키지

![img](../Img/k8s_56.png)<br>

> elinks 생성된 서버 IP

![img](../Img/k8s_57.png)<br>
터미널에서 웹 브라우저를 볼 수 있다.

> kubectl create deployment mainui --image=httpd --replicas=3

create : 여러개를 만들때 사용한다.<br>
httpd 서버를 사용하고 mainui라는 이름의 웹서버를 3개 만든다.<br>

![img](../Img/k8s_59.png)<br>
node1과 node2에 k8s가 알아서 제어하며 생성한다.<br>

![img](../Img/k8s_62.png)<br>

> kubectl edit deployments.apps mainui

pod 환경을 설정할 수 있다.<br>

![img](../Img/k8s_63.png)<br>

> kubectl run webserver --image=nginx:1.14 --port 80 --dry-run -o yaml > webserver-pod.yaml

--dry-run은 실행이 되는지 테스트를 한다.<br>

-o yaml > webserver-pod.yaml: 실행이 된다면
해당 pod의 정보를 저장하는 yaml파일을 만든다.<br>

![img](../Img/k8s_64.png)<br>

> kubectl delete pod webserver

pod를 삭제한다.

![img](../Img/k8s_65.png)<br>

> kubectl create -f webserver-pod.yaml

방금 생성한 yaml파일을 컨테이너 pod로 생성한다.<br>

### [컨테이너 내부 제어]

![img](../Img/k8s_60.png)<br>

> kubectl exec webserver -it -- /bin/bash

![img](../Img/k8s_61.png)<br>

> kubectl port-forward webserver 8080:80

포트포워드도 된다.<br>





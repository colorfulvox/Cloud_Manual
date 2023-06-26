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

관리자 게정으로 재로그인 하면 잘 된다.<br>

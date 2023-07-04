# k8s\_livenessProbe

[뒤로가기](../)\


## \[livenessProbe] 컨테이너 진단

k8s에서는 컨테이너에대한 self-healing 기능을 제공한다.\
그리고 livenessProbe가 k8s의 self-healing 기능이다.\
livenessProbe는 Pod가 계속 실행할 수 있음을 보장한다.\


#### \[기존 nginx pod yaml]

```
apiVersion: v1
kind: Pod
metadata:
  name: nginxserver
spec:
  containers:
  - name: nginx
     image: nginx:1.14
```

#### \[livenessProbe가 추가된 nginx yaml]

```
apiVersion: v1
kind: Pod
metadata:
  name: nginxserver
spec:
  containers:
  - name: nginx
     image: nginx:1.14
     livenessProbe:
        httpGet:
             path: /
             port: 80
```

livenessProbe를 통해서 건강 검진을 하도록 요청한다. (http 80port를 통해 검진한다.)

### livenessProbe가 지원하는 probe

#### \[httpGet]

지정한 IP주소, port, path에 HTTP GET 요청을 보내 해당 컨테이너가 응답하는지 확인\
200이 아닌 값이 나오면 오류로 판단해 컨테이너를 다시 시작함\


```
livenessProbe:
    httpGet:
         path: /
         port: 80
```

#### \[tcpSocket]

지정한 port에 TCP연결을 시도\
연결되지 않으면 컨테이너를 다시 시작함\


```
livenessProbe:
    tcpSocket:
       port: 22
```

#### \[exec]

exec 명령을 전달\
명령의 종료코드가 0이 아니면 컨테이너를 다시 시작함\


```
livenessProbe:
     exec:
         command:
         - ls
         - /data/file
```

#### 예시

사용하는 pod.yaml 파일

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-liveness
spec:
  containers:
  - name: nginx-container
    image: nginx:1.14
    ports:
    - containerPort: 80
      protocol: TCP
    livenessProbe:
      httpGet:
        path: /
        port: 80
```

![img](../Data/Img/k8s\_liveness1.png)

> kubectl create -f nginx-liveness.yaml

yaml파일에 대한 Pod를 실행한다.

![img](../Data/Img/k8s\_liveness2.png)

> kubectl describe pod nginx-liveness

실제로 Pod를 자세히 보면 Liveness가 동작했던것을 확인할 수 있다.

#### 추가적인 기능

> delay=0s timeout=1s period=10s #success=1 #failure=3

initialDelaySeconds: 0 컨테이너가 Running 후 언제 검사할지 Delay를 조정\
timeoutSeconds: 1 검사를 위해 Get요청을하고 몇초를 기다릴지 조정 가능\
periodSeconds: 20 몇초마다 검사를 진행할지 조정\
successThreshold: 1 요청에 대한 성공 횟수를 조정 \[해당 성공 횟수가 충족되야 검사 완료]\
failureThreshold: 3 요청에 대한 실패 횟수를 조정 \[해당 실패 횟수가 충족되야 검사 실패]\


이런 추가적인 기능에대한 자세한 정보는\


![img](../Data/Img/k8s\_liveness3.png)

> kubectl get pod nginx-liveness -o yaml

pod를 yaml형식으로 확인하면 된다.\


#### 실패 예시

예시에 쓰일 pod.yaml 파일

```
apiVersion: v1
kind: Pod
metadata:
  name: unhealthy-pod
spec:
  containers:
  - name: unhealthy-container
    image: smlinux/unhealthy
    ports:
    - containerPort: 80
      protocol: TCP
    livenessProbe:
      httpGet:
        path: /
        port: 8080
```

smlinux/unhealthy 이미지는 처음 5번은 정상적인 상태 코드를 응답하고\
그 뒤로 HTTP 500 ERROR를 발생시키는 이미지이다.\


![img](../Data/Img/k8s\_liveness4.png)

> kubectl create -f unhealthy-pod.yaml

![img](../Data/Img/k8s\_liveness5.png)

> kubectl describe pod unhealthy-pod

3번 실패해 컨테이너를 삭제하고 다시 생성하는 것을 볼 수 있다.\

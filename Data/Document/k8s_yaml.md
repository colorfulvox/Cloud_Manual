[뒤로가기](../../README.md)<br>

# yaml 템플릿

쉽게 읽을 수 있는 데이터 직렬화 양식<br>
python처럼 들여쓰기로 데이터 계층을 표기<br>
(key-value타입)
배열 문법 : '-' 문자로 여러개 나열 가능

k8s object를 정의할때 apiVersion이 필요하다.

API Object 종류<br>
Deployment -> apps/v1<br>
Pod -> v1<br>
ReplicaSet -> apps/v1<br>
ReplicationController -> v1<br>
Service -> v1<br>
PersistentVolume -> v1

예시)

pod yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  namespace: orange
spec:
  containers:
  - image: nginx:1.14
    name: nginx
    ports:
    - containerPort: 80
    - containerPort: 443
```

deployment yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: testdeploy
  name: testdeploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: testdeploy
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - image: httpd
        name: httpd
```

[뒤로가기](../../README.md)<br>

<<<<<<< HEAD
[뒤로가기](../../)\
=======
# infra container (pause)
>>>>>>> parent of 4d4a984 (GITBOOK-5: change request with no subject merged in GitBook)

Pod를 생성하면 pause 컨테이너가 자동으로 생성된다.<br>
pause 컨테이터는 해당 Pod에대한 ip, 호스트등을 관리해주는 컨테이너이다.<br>

## 예시

<<<<<<< HEAD
Pod를 생성하면 pause 컨테이너가 자동으로 생성된다.\
pause 컨테이터는 해당 Pod에대한 ip, 호스트등을 관리해주는 컨테이너이다.\


### 예시

![img](../Img/k8s\_infra\_container1.png)
=======
![img](../Img/k8s_infra_container1.png)
>>>>>>> parent of 4d4a984 (GITBOOK-5: change request with no subject merged in GitBook)

> kubectl run infratest --image:nginx:1.14 --port=80

nginx이미지를 사용하는 Pod를 생성했다.

<<<<<<< HEAD
![img](../Img/k8s\_infra\_container2.png)
=======
![img](../Img/k8s_infra_container2.png)
>>>>>>> parent of 4d4a984 (GITBOOK-5: change request with no subject merged in GitBook)

> kubectl get pod infratest -o wide

Pod를 확인하면 node2에서 실행한다.<br>
node2에 들어가본다.<br>

<<<<<<< HEAD

![img](../Img/k8s\_infra\_container3.png)
=======
![img](../Img/k8s_infra_container3.png)
>>>>>>> parent of 4d4a984 (GITBOOK-5: change request with no subject merged in GitBook)

> ctr --namespace k8s.io containers list

실제로 node2에 pause들이 생성되있는것을 볼 수 있다.

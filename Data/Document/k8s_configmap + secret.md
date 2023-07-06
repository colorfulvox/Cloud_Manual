[뒤로가기](../../README.md)<br>

# ConfigMap

컨테이너 구성 정보를 한곳에 모아서 관리하는 API

컨테이너가 많을 경우 하나하나 관리하는것은 쉽지않다.<br>
그래서 ConfigMap을 활용해서 모든 컨테이너의 구성 정보를 관리한다.<br>

key-value값으로 관리를 하며
file도 configmap에 저장할 수 있다.

## ConfigMap 생성 방법

### kubectl create configmap NAME [--from-literal=key1=value1]

### kubectl create configmap NAME [--from-file=source]

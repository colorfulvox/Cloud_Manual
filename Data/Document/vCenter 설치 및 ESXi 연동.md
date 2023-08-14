[뒤로가기](../../README.md)<br>

# vCenter 설치 및 ESXi 연동

![img](../Img/vmware115.png)<br>
현재 ESXi에 vCenter Server가 없다.<br>
그래서 vCenter Server를 설치한뒤 연동을 진행한다.<br>

## 시작

### (1) vCenter 설치

![img](../Img/vmware116.png)<br>

### [vSphere 8](https://customerconnect.vmware.com/en/evalcenter?p=vsphere-eval-8)

해당 링크에 들어간뒤 vCenter 8를 다운받는다.<br>
참고로 vCenter는 ESXi 현재 버전(7)과 같거나 높아야 된다.<br>
그래서 여기서는 vCenter 8 버전을 설치한다.<br>

![img](../Img/vmware117.png)<br>
다운이 완료되면 해당 ISO파일을 클릭한다.<br>

![img](../Img/vmware118.png)<br>
vcsa-ui-installer로 들어간다.<br>

![img](../Img/vmware119.png)<br>
각 호스트PC의 환경에 맞는 폴더로 들어간다.<br>

![img](../Img/vmware120.png)<br>
installer를 누른다.<br>

![img](../Img/vmware121.png)<br>
설치 프로그램이 실행되면, 언어를 바꾸고 싶으면 바꾸고
설치를 누른다.<br>

![img](../Img/vmware122.png)<br>
다음을 누른다.<br>

![img](../Img/vmware123.png)<br>
라이센스 체크를 누르고 다음을 누른다.<br>

![img](../Img/vmware124.png)<br>
ESXi의 IP를 입력하고 ESXi의 사용자 이름, 암호를 입력한다.<br>

![img](../Img/vmware125.png)<br>
vCenter의 이름과 암호를 입력하고 다음으로 넘어간다.<br>

![img](../Img/vmware126.png)<br>

현재 vCenter의 최소 리소스를 보면<br>
CPU2, 14GB메모리, 578GB 저장소 이다.<br>

### 하지만 문제는 현재 실습하는 환경이 578GB를 가진 디스크가 없다.

그래서 실습이 중단될 수 있지만, 계속 설치를 진행한다.<br>

![img](../Img/vmware127.png)<br>

vCenter를 설치할 데이터스토어를 지정할 수 있다.<br>
연결된 ESXi에 설치하는 것이다.<br>
여기서는 새로운 데이터스토어를 생성해 저장소를 할당하겠다.<br>

![img](../Img/vmware128.png)<br>
혹시 모르니 기존의 ISO, Ubuntu Storage를 모두 지우고 120GB 크기로 할당했다.
<br>

![img](../Img/vmware129.png)<br>
또한, 씬 디스크 모드를 사용한다.<br>
(실제 사용하는 용량만 할당한다. 테스트 환경일때 사용한다.)<br>

![img](../Img/vmware130.png)<br>
현재 vCenter는 ESXi에서 실행된다.<br>
그래서 IP주소도 ESXi IP주소에 맞춰 구성했다.<br>
ex) 192.168.80.x<br>

또한, 기본 게이트웨이와 DNS 서버도 ESXi의 네트워크 구성에 맞춰 구성했다.<br>

실제는 관리자 네트워크이기에 구성이 복잡할 수 있다.<br>

![img](../Img/vmware131.png)<br>
구성한 환경을 검토하고 완료를 누른다.

![img](../Img/vmware132.png)<br>
설치를 진행한다.<br>

![img](../Img/vmware133.png)<br>
설치가 완료되면 두번째 설치 단계로 넘어가게 된다.<br>
계속을 누른다.<br>

![img](../Img/vmware134.png)<br>
vCenter server 설정을 진행한다.<br>
다음으로 넘어간다.<br>

![img](../Img/vmware135.png)<br>
별다른 NTP 서버가 없으니 ESXi 호스트와 시간 동기화를 하고<br>
SSH 연결을 활성화 한다.<br>

![img](../Img/vmware136.png)<br>
실제 vCenter에 로그인할때 필요한 계정을 생성한다.<br>
여기서는 기본값인 도메인 이름을 사용했다.<br>

![img](../Img/vmware137.png)<br>
다음으로 넘어간다.<br>

![img](../Img/vmware138.png)<br>
환경 구성을 검토하고 완료한다.<br>

![img](../Img/vmware139.png)<br>
최종 환경 설정을 구성한다.<br>

![img](../Img/vmware140.png)<br>
설치가 완료되면 해당 링크로 접속한다.<br>

![img](../Img/vmware141.png)<br>
VSPHERE CLIENT 시작을 누른다.<br>

![img](../Img/vmware142.png)<br>

> administrator@vsphere.local

아까 생성한 관리자 계정명이다.<br>
비밀번호도 입력하고 로그인을 한다.<br>

![img](../Img/vmware143.png)<br>
설치가 완료되었다.<br>

### (2) ESXi 연동

![img](../Img/vmware144.png)<br>
vCenter를 사용하려면 먼저 데이터센터를 생성해야된다.<br>
새 데이터 센터를 누른다.<br>

![img](../Img/vmware145.png)<br>
기본값으로 두고 확인을 눌러 생성한다.<br>

![img](../Img/vmware146.png)<br>
생성된 데이터센터를 오른쪽 클릭한뒤,
호스트 추가를 클릭한다.<br>

![img](../Img/vmware147.png)<br>
연동할 ESXi 호스트의 IP주소를 입력하고 다음으로 넘어간다.<br>

![img](../Img/vmware148.png)<br>
ESXi에 접속이 가능한 계정과 암호를 입력하고 다음으로 넘어간다.
<br>

![img](../Img/vmware149.png)<br>
ESXi 호스트의 구성정보를 확인할 수 있다.<br>
다음으로 넘어간다.<br>

![img](../Img/vmware150.png)<br>
Host lifecycle은 ESXi 호스트의 소프트웨어 설치 및 업데이트를 자동으로 진행하며 관리하는 기능이다.<br>
체크를 하고 다음으로 넘어간다.<br>

![img](../Img/vmware151.png)<br>
Host lifecycle로 새 이미지를 선택하게 한다.<br>
기존 이미지로 유지한다.<br>
현재 설치된 ESXi는 7.0 U3n이다.<br>
찾아서 해당 버전에 맞게 설정한다.<br>

![img](../Img/vmware152.png)<br>
평가판을 쓰고 있으니 다음으로 넘어간다.<br>

![img](../Img/vmware153.png)<br>
잠금 모드가 필요없으니 사용 안함으로 하고 다음으로 넘어간다.<br>

![img](../Img/vmware154.png)<br>
방금 생성한 데이터센터를 선택하고 다음으로 넘어간다.<br>

![img](../Img/vmware155.png)<br>
구성 정보를 검토하고 완료한다.<br>

![img](../Img/vmware156.png)<br>
이렇게 ESXi가 vCenter랑 연결되었다.

![img](../Img/vmware157.png)<br>
실제로 ESXi 호스트의 vSphere Client로 들어가면
연결되었다 표시된다.<br>

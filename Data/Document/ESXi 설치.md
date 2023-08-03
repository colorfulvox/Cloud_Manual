[뒤로가기](../../README.md)<br>

# ESXi 설치

ESXi 7.0을 설치한다.<br>

[하드웨어의 최소 요구 사항]<br>

- CPU 2 Core
- BIOS CPU -> NX/XD 비트 사용 허용
- 일반적인 운영 환경에서 가상 시스템을 실행하려면 최소 8GB메모리
- 최소 32GB 저장소 (하지만 권장 사양인 142GB로 할것이다.)
- CPU 가상화 설정 [Intel VT-x]

## 시작

### (1) ESXi ISO 파일 다운로드

### [VMware 사이트](https://www.vmware.com/kr.html)

ESXi ISO파일을 다운받기 위해서는 VMware 회원가입을 해야한다.<br>
회원가입을 하고 진행한다.<br>

### [vSphere 7.0](https://customerconnect.vmware.com/en/evalcenter?p=free-esxi7)

![img](../Img/vmware21.png)<br>
회원가입을 진행한뒤 해당 사이트로 들어가 vSphere 7.0 제품 등록울 진행한다.<br>

![img](../Img/vmware22.png)<br>
인적 사항을 작성한뒤 등록한다.<br>

![img](../Img/vmware23.png)<br>
등록이 완료되면 다운로드 항목이 뜨며 ISO파일을 다운로드 받는다.<br>

### (2) ESXi VM 구성

![img](../Img/vmware24.png)<br>
VMware Workstation을 실행한뒤 Create a New Virtual Machine을 클릭한다.<br>

![img](../Img/vmware25.png)<br>
Custom을 선택한뒤 다음으로 넘어간다.<br>

![img](../Img/vmware26.png)<br>
ESXi 7.0을 선택하고 다음으로 넘어간다.<br>

![img](../Img/vmware27.png)<br>
다운받은 ISO파일을 등록하고 다음으로 넘어간다.<br>

![img](../Img/vmware28.png)<br>
원하는 이름과 경로를 선택한뒤 다음으로 넘어간다.<br>

![img](../Img/vmware29.png)<br>
CPU 설정이다. 이대로 두고 다음으로 넘어간다.<br>

![img](../Img/vmware30.png)<br>
메모리 크기 설정이다.<br>
여기서는 8GB를 할당했다.<br>

![img](../Img/vmware31.png)<br>
네트워크 설정 부분이다.<br>
지금은 기본인 NAT으로 설정하지만 추후에 필요할경우 어댑터를 추가할 수 있다.<br>

Bridge : 공유기로부터 IP를 할당 받아 호스트 PC와 돌일한 네트워크 대역의 IP를 갖게 된다.<br>
(공유기를 통해 외부 네트워크와 통신)<br>

NAT : 호스트 PC로부터 IP를 할당 받아 자체 DHCP 서버를 띄워
내부 네트워크 대역 할당 및 통신을 한다.<br>
(호스트PC를 통해 외부 네트워크와 통신)<br>

Host-Only : 내부 네트워크를 구축한다.<br>
(VM끼리만 통신이 가능)<br>

![img](../Img/vmware32.png)<br>
SCSI 컨트롤러는 VMware 반가상화 컨트롤러로 처리량을 높이고 CPU 사용량을 낮추는 고성능 스토리지 컨트롤러이다.<br>
또한, ESXi 4.x 이상의 가상 시스템과 호환되기에 이대로 설정한다.<br>

![img](../Img/vmware33.png)<br>
디스크 타입도 SCSI로 두고 다음으로 넘어간다.<br>

![img](../Img/vmware34.png)<br>
가상 디스크 설정 부분이다.<br>

Create a new virtual disk : 가상 디스크 새로 생성<br>
Use an existing virtual disk : 기존 가상 디스크 사용<br>
Use a physical disk : 물리적 디스크 사용<br>

가상 디스크를 새로 생성한다.<br>

![img](../Img/vmware35.png)<br>
저장소 크기 설정이다.<br>

Allocate all disk space now 선택시 디스크의 성능을 증가시키지만,<br>
설정한 디스크의 용량이 한번에 모두 할당되어 호스트 PC의 디스크 용량이 부족할 수 있다.<br>

Store virtual as a single file : 하나의 가상 디스크 파일로 사용하는 설정<br>

Split virtual into mulitple files : 여러 디스크 파일로 분할한다.<br>
이동 및 관리가 편해지기에 멀티 파일로 선택한다.<br>

![img](../Img/vmware36.png)<br>
디스크 이름 설정이다.<br>
이대로 둘 것이다.<br>

![img](../Img/vmware37.png)<br>
최종 설정 검토이다.<br>
확인한뒤 Finish를 누른다.<br>

![img](../Img/vmware38.png)<br>
그럼 자동으로 ESXi 설치 구성을 진행한다.<br>

### (3) ESXi 설치 및 구성

![img](../Img/vmware39.png)<br>
Enter를 누른다.<br>

![img](../Img/vmware40.png)<br>
라이센스 관련 내용이다. F11을 누른다.

![img](../Img/vmware41.png)<br>
설치를 진행할 저장소를 선택하는 부분이다.<br>
Enter를 누른다.<br>

![img](../Img/vmware42.png)<br>
키보드 언어 설정이다.<br>
그대로 두고 Enter를 누른다.<br>

![img](../Img/vmware43.png)<br>
비밀번호를 설정하고 Enter를 누른다.<br>

![img](../Img/vmware44.png)<br>
F11를 눌러 설치를 진행한다.<br>

![img](../Img/vmware45.png)<br>
설치가 완료되면 Enter를 눌러 재시작한다.<br>

![img](../Img/vmware46.png)<br>
ESXi가 실행되면 해당 부분에 웹 인터페이스 URL를 제공한다.<br>

![img](../Img/vmware47.png)<br>
해당 URL를 복사해 웹을 통해 접속한뒤 아까 설정한 ESXi root계정으로 접속한다.<br>

![img](../Img/vmware48.png)<br>
접속 완료 화면이다.<br>

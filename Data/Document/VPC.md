[뒤로가기](../../README.md)

# VPC 생성

![img](../Img/VPC1.png)<br>
(AWS에서 VPC검색후 접속)<br>

![img](../Img/VPC2.png)<br>
(VPC생성 클릭)<br>

![img](../Img/VPC3.png)<br>
(VPC만 체크하고 이름, IPv4 CiDR는 다음과 같이 입력하고 생성 클릭)<br>

## 서브넷 생성

![img](../Img/VPC4.png)<br>
(서브넷 생성 클릭)<br>

![img](../Img/VPC5.png)<br>
(방금 생성한 VPC, 가용 영역을 선택, 서브넷 이름과 IPv4 CiDR 입력한뒤 서브넷 생성 클릭)<br>

![img](../Img/VPC6.png)<br>
(private 서브넷도 생성)

![img](../Img/VPC7.png)<br>
-> 생성한 서브넷에게는 AWS가 NACL와 라우팅 테이블을 연결시켜 준다. 이제 명시적으로 연결 시켜주자. <br>

### 라우팅 테이블 생성

![img](../Img/VPC8.png)<br>
(작업에서 서브넷 연결 편집을 누른다.)<br>

![img](../Img/VPC9.png)<br>
(private만 선택하고 연결한다.)<br>

![img](../Img/VPC10.png)<br>
(public은 따로 라우팅 테이블을 생성해 연결하자. 라우팅 테이블을 생성한다.)<br>

![img](../Img/VPC11.png)<br>
(이름과 생성한 VPC를 선택하고 라우팅 테이블을 생성한다.)<br>

![img](../Img/VPC12.png)<br>
(생성한 라우팅 테이블에 Public 서브넷을 연결시켜주자)<br>

-> public 서브넷은 외부와 연결이 되어야하므로 인터넷 게이트웨이를 연결 시켜줘야 한다.

### 인터넷 게이트웨이 생성

![img](../Img/VPC13.png)<br>
(인터넷 게이트웨이 생성 클릭)<br>

![img](../Img/VPC14.png)<br>
(이름을 입력후 생성을 클릭)<br>

![img](../Img/VPC15.png)<br>
(VPC연결을 누르거나 작업에서 VPC 연결을 클릭한다.)<br>

![img](../Img/VPC16.png)<br>
(생성한 VPC를 선택한후 연결을 클릭)<br>

![img](../Img/VPC17.png)<br>
(라우팅 테이블에서 생성한 라우팅을 선택하고 라우팅 편집을 클릭)<br>

![img](../Img/VPC18.png)<br>
(라우팅 추가를하고 모든 IP 접근이 가능하게 한뒤 대상에서 "인터넷 게이트웨이"를 검색하면 <br>방금 생성한 게이트웨이가 선택에서 뜬다. 선택한뒤 저장을 한다.)<br>

### 테스트

### Public

![img](../Img/VPC19.png)<br>
(EC2를 새로 생성할때 네트워크 설정에서 생성한 VPC, 서브넷은 생성한 public 서브넷으로 선택한다.)

![img](../Img/VPC20.png)<br>
(생성이 되었지만 보안 그룹이 Default여서 아직 연결이 안된다.
보안 그룹을 수정하자.)

![img](../Img/VPC21.png)<br>
(기존 규칙을 삭제하고 다음과같이 설정해 저장하자)<br>

![img](../Img/VPC22.png)<br>
(이제 EC2에 접속한뒤)

> curl -v naver.com

을 입력하자 다음과 같이 외부로 통신이 되는것을 볼 수 있다.<br>

### Private

![img](../Img/VPC23.png)<br>
(이번엔 키페어도 연결해주고 퍼블릭 IP 자동 할당도 Private이기에 필요없어 비활성화를 해준다.)<br>
[private 서브넷으로 설정해야한다.]<br>

![img](../Img/VPC24.png)<br>
(Private ec2는 연결이 불가능하다. <br>만약 퍼블릭 ip를 활성화 했다해도 연결이 불가능하다.)<br>

-> 그래서 Bastion Host로 public EC2를 통해 접속해보자.
<br>
![img](../Img/VPC25.png)<br>
(아까 생성한 키페어를 복사한다.)<br>

![img](../Img/VPC26.png)<br>
(아까 생성한 public ec2로 접속해 키페어를 생성한뒤)

![img](../Img/VPC27.png)<br>
(복사한 키페어 내용을 붙여넣어 저장한다.)<br>

![img](../Img/VPC28.png)<br>
(키페어의 권한을 부여한다.)

![img](../Img/VPC29.png)<br>
(private ec2의 프라이빗 ip주소를 복사한다.)<br>

![img](../Img/VPC30.png)<br>
public ec2에서<br>

> ssh -i "생성한키페어.pem" ec2-user@프라이빗 ip주소

를 입력해 접속한다.<br>

![img](../Img/VPC31.png)<br>
(연결을 했지만 외부로 접속이 안되는것을 볼 수 있다.)<br>

-> NAT 게이트웨이를 통해서 외부로 접속할 수 있도록 설정하자.<br>

![img](../Img/VPC32.png)<br>
(VPC에서 NAT 게이트웨이 생성을 클릭)<br>

![img](../Img/VPC33.png)<br>
(이름을 입력하고 서브넷같은 경우 퍼블릭을 통해서 접근하기때문에 생성한 public 서브넷으로 선택한다. <br>그리고 탄력적 IP가 없으면 할당을 해준다.)<br>

![img](../Img/VPC34.png)<br>
(현재 private은 외부로 가는 통로가 없다.
그래서 private 라우팅 테이블에서 통로를 추가해주자.)

![img](../Img/VPC35.png)<br>
(0.0.0.0/0으로 모든 경로로 설정하고 대상에서 "NAT 게이트웨이"를 검색하면<br> 방금 생성한 NAT이 목록에 뜬다.
선택한뒤 저장한다.)<br>

![img](../Img/VPC36.png)<br>
(private ec2도 public을 거쳐 외부로 연결이 되는 것을 볼 수 있다.)<br>



[가상 데이터 센터 만들기 - VPC 기본 및 연결 옵션 - 양승도 솔루션즈 아키텍트(AWS 코리아)](https://www.youtube.com/watch?v=R1UWYQYTPKo)

위 영상을 정리한다.

# 가상 데이터 센터 만들기

AWS 에서 리소스의 위치를 결정지을 수 있는 기본이 VPC. 서비스의 성격에 맞는 네트워크에 연결해야 원하는 고객들에게 서비스를 제공할 수 있다.

과거의 AWS 에서는 EC2 를 생성하면 그 위치를 지정할 수 없었다. 커다란 AWS 클라우드 내에 여러 고객의 인스턴스가 하나의 네트워크에 구성되던 시절이 있었다.

### VPC(Virtual Private Cloud)

특정 고객 전용의 가상 데이터 센터. VPC 를 사용하지 않던 시절에는 `EC2 Classic` 이라고 불렀다. 지금은 EC2 Classic 환경을 제공하지 않고 VPC 환경만을 제공하고 있다.

VPC 에서는 내부적 통신을 위해서 private IP address 를 사용하게 된다. EC2 인스턴스가 터미네이트 되기 전 까지 IP 를 할당할 수 있다. 서브넷 단위로 공간을 분리해 가상 머신을 위치시켜 사용하게 된다. 어떤 서브넷은 외부와 연결하게 할 수 있다. 예를 들어 데이터베이스가 위치한 서브넷에서는 외부와 통신을 연결하지 않는다.

![image](https://user-images.githubusercontent.com/39546083/122400058-527f3d00-cfb6-11eb-968f-739c8468cc80.png)

어떤 절차와 컴포넌트로 구성할 수 있는지 설명한다.

- VPC 개념에 익숙해지기
- 기본 VPC 설정 살펴보기
- 요구에 맞게 가상 네트워크를 설정할 수 있는 방법 알아보기 → 지금 사용하는 온 프레미스 데이터 센터 또는 다른 VPC 와 연결할 수 있다.

## 연습: 인터넷에 연결된 VPC 설정

![image](https://user-images.githubusercontent.com/39546083/122400389-a38f3100-cfb6-11eb-97b5-9e2989e1800b.png)

위 네 가지 단계를 따른다.

- 내부적으로 사용하는 private IP 주소를 선택한다.
- 서브넷 단위로 바운더리를 만들어준다.
- 인터넷과 통신할 수 있는 라우팅 테이블을 만든다.
- 접속 허용 / 차단에 대한 방화벽 설정을 한다.

### IP 주소 범위 선택

`CIDR` 를 사용한다. IP 주소를 `classless` 방식으로 설정한다.

IPv4 의 주소 체계를 갖는다.

![image](https://user-images.githubusercontent.com/39546083/122400778-fec12380-cfb6-11eb-9b03-38cd7c3906e1.png)

위 예시를 보면 16 이므로 앞의 16비트는 항상 고정이고 나머지 비트를 가변적으로 사용할 수 있다라는 뜻이다. 즉 `2^16` 개 만큼의 IP 주소를 할당할 수 있는 것이다.

![image](https://user-images.githubusercontent.com/39546083/122401316-70996d00-cfb7-11eb-82a9-b5e93f7405b3.png)

RFC1918 에 명시된 IP 주소를 VPC 의 private address 로 사용할 것을 권장한다. 작은 네트워크를 사용할 필요가 없다면 16이라는 마스킹 값을 갖는 많은 워크로드를 운용할 수 있다. 이후에 VPC 는 데이터 센터와 연결하고 다른 VPC 와 연결해야할 것이다. 연결할 곳의 주소가 나의 주소와 겹치게 되면 찾아가는 길을 잃을 수 있다. overlapping 이 되지 않는 완전히 분리된 주소를 사용해야 한다.

공인 주소가 아닌 private address 를 사용할 때는 RFC 1918 에 명시된 표준을 따르는 것을 권장한다. 여기에 명시된 네트워크를 사용해야 한다. 다른 주소를 사용하게 된다면 AWS 의 다른 VPC 들과 연결이 용이하지 않을 수 있다.

### 서브넷

VPC 내의 작은 공간. AWS 는 리전 내에 복수 개의 AZ 를 제공하고 있다. VPC 는 AZ 와 상관 없이 리전 단위로 큰 네트워크를 만들 수 있다. 서브넷은 AZ 안에 각각 만들어야 한다.

```
VPC > AZ > Subnet
```

서브넷은 중복되지 않아야 한다. 서브넷 간에 경로를 찾아갈 때(라우팅을 할 때) 네트워크가 겹치면 안된다.

![image](https://user-images.githubusercontent.com/39546083/122401961-06cd9300-cfb8-11eb-9f7b-3f80f813f3ec.png)

![image](https://user-images.githubusercontent.com/39546083/122402021-13ea8200-cfb8-11eb-8442-3f02542f1b4b.png)

서브넷은 24 마스크를 갖는 것을 권장한다. 256개가 아닌 251개를 사용할 수 있다. AWS 가 앞에 4개, 뒤에 1개를 관리의 목적으로 리저빙한다.

### 인터넷 경로

![image](https://user-images.githubusercontent.com/39546083/122402243-43998a00-cfb8-11eb-8da6-14eef6b186ce.png)

VPC 를 만들면 기본적인 라우팅 테이블이 생성된다. 하나의 라우팅 테이블을 모든 서브넷에 동일하게 적용할 필요 없이 커스텀 할 수 있다.

![image](https://user-images.githubusercontent.com/39546083/122402390-64fa7600-cfb8-11eb-9ca1-36d046a18e85.png)

- 위 destination 은 VPC 가 사용하는 주소. VPC 내부에서 패킷을 전달하겠다는 로컬 목적지를 명시한다.

**Internet Gateway**

Gateway: 대문 → Internet Gateway: 인터넷과 연결되는 문을 정의하는 것

![image](https://user-images.githubusercontent.com/39546083/122402782-c6224980-cfb8-11eb-9cef-3b73e3236b8b.png)

- 0.0.0.0/0: 지구상에 존재하는 모든 네트워크. `anywhere`
- 로컬로 보내는 패킷이 아닌 다른 네트워크 범위로 패킷을 보내면 `igw` 라는 곳으로 패킷을 보내게 된다.
- 그 후에 인터넷에 존재하는 네트워크끼리 패킷을 주고받게된다.

### VPC 의 네트워크 보안: Network ACLs / Security Groups

기본적으로 설정하는 보안 정책. AWS 는 사용자가 정의하기 전에 모든 요청을 불허하는 정책을 만들어 준다.

방화벽 기능은 크게 두가지가 있다.

**Network ACLs: Stateless firewalls**

서브넷 단위로 적용하는 방화벽. `Stateless` 로 동작한다.

![image](https://user-images.githubusercontent.com/39546083/122403586-72fcc680-cfb9-11eb-998e-770eb64326da.png)

- 위 규칙을 보면 모든 트래픽을 허용하도록 설정되어 있다.

**Security groups: 애플리케이션 구조**

![image](https://user-images.githubusercontent.com/39546083/122403892-b22b1780-cfb9-11eb-911f-a9eb0014f99b.png)

- 위와 같은 애플리케이션 구조를 많이 생각한다.
- 가장 앞단의 애플리케이션 서버(웹 서버)는 모든 트래픽을 수용한다.
- 백엔드 서버는 애플리케이션 서버의 트래픽만 허용하도록 설정한다.

→ AWS 에서 권장하는 방식.

웹 서버의 SG

![image](https://user-images.githubusercontent.com/39546083/122404166-f3232c00-cfb9-11eb-9d51-b07b82247285.png)

백엔드 서버의 SG

![image](https://user-images.githubusercontent.com/39546083/122404254-0635fc00-cfba-11eb-93f2-4d0b90a3e75c.png)

- TCP 2345 포트로 요청을 받고 있다.
- Source 를 IP 주소로 명시해도 되고 sg 를 명시해도 된다.
- Source 에 sg 를 명시하는 것의 장점은 오토 스케일링할 때 직접 모든 IP 를 지정하지 않아도 된다는 것이다.

→ 원하는 포트에서 원하는 주소로 들어오는 트래픽만 허용할 수 있다.

**Stateful vs Stateless**

Stateful: 인바운드 트래픽(들어오는 트래픽)에 대해 커넥션을 그대로 기억한다. 웹 서버가 프로세싱 후 응답을 보낼 때 해당 채널로 보낸다. 나갈때는 별도의 설정 없이 트래픽을 내보낼 수 있다. 즉, 아웃바운드 트래픽(나가는 트래픽)에 대한 규칙을 지정하지 않아도 된다.

Stateless: 들어오는 트래픽에 대한 기억을 보존하지 않는다. 클라이언트는 나가는 포트를 명확하게 지정할 수 없다. OS 마다 포트를 나눠서 사용하고 있기 때문에 나가는 포트에 대한 규칙을 알아야 한다. 들어오는 포트와 나가는 포트를 쌍으로 지정하지 않으면 요청을 처리할 수 없다.

웰론 포트: 1024번 이하의 포트들. OS 의 데몬에서 사용하는 포트.

1025 ~ 65536: 클라이언트가 트래픽을 내보내는 포트로 랜덤하게 사용한다.

Security Group 을 생성할 때는 아래 규칙을 따라야 한다.

- 최소 권한 원칙(Prinicple of Least Privilege) 준수: 방화벽 내부로 진입할 수 있는 IP 는 사내의 관리자의 계정으로만 접속 가능하도록 설정한다.
- VPC 는 egress / ingress 에 대한 Security Group 생성 가능: 인바운드 / 아웃바운드, egress / ingress 를 따로 설정 가능하다.

## VPC 의 연결 옵션

### 인터넷 액세스 제한: 서브넷 별로 다른 라우팅

서브넷 별로 각기 다른 라우팅이 필요하다.

![image](https://user-images.githubusercontent.com/39546083/122405629-2adea380-cfbb-11eb-8463-3211c6450a66.png)

![image](https://user-images.githubusercontent.com/39546083/122406001-7abd6a80-cfbb-11eb-8a11-743251044f32.png)

- 위 그림을 봤을 때 오른쪽 서브넷은 외부 인터넷과 연결된 서브넷이다.
- 이때 문제가 왼쪽의 프라이빗 서브넷 내부의 애플리케이션 패치, OS 패치 등을 수행해야 할 때 외부 서버의 리포지토리와 연결해야 하는 문제가 생긴다.
- 이때 사용하는 것이 `NAT` 를 사용해야 한다. → Network Address Translate
- 장점이 왼쪽의 프라이빗 서브넷은 외부 레포지토리와 통신이 가능한 반면 외부 인터넷은 왼쪽의 프라이빗 서브넷에 접근할 수 없다 → 프라이빗 서브넷은 `public IP` 주소가 없기 때문이다.

![image](https://user-images.githubusercontent.com/39546083/122406277-af312680-cfbb-11eb-905a-8c7aeab0886f.png)

### VPC 간 연결: VPC peering

![image](https://user-images.githubusercontent.com/39546083/122406568-ea335a00-cfbb-11eb-8b03-d82ed69f6c5e.png)

- `공통 모듈`을 만들어서 사용하는데 공통 모듈은 아주 private 하게 관리하고 싶을 때 사용한다.
- VPC peering 을 하게 되면 다른 VPC 의 SG 를 레퍼런싱할 수 있어서 보안 정책을 강화할 수 있다.
- VPC peering 은 AWS 계정이 다를 수 있음. 피어링 요청 → 피어링 수락 과정이 필요하다.
- 경로를 생성해야 한다.

![image](https://user-images.githubusercontent.com/39546083/122406945-35e60380-cfbc-11eb-9175-d4b39aa83eed.png)

- pcx: 피어링 게이트웨이.
- 현재 VPC 피어링의 제약사항이 있다. → Transit VPC 를 지원하지 않는다.
- Transit VPC 란 VPC C ↔ VPC A ↔ VPC D 일 때 VPC C ↔ VPC D 가 가능하게 만들어 지는 것을 말한다.
- 불필요한 피어링 게이트웨이는 만들지 않는다.

## 회사 네트워크에 연결: Virtual Private Network(VPN) & Direct Connect(DX)

회사의 보안이 필요한 데이터를 전송하고 싶을 때 사용한다.

`하이브리드 네트워크 구조` 라고 한다.

![image](https://user-images.githubusercontent.com/39546083/122408009-01267c00-cfbd-11eb-9143-de1d537a3db5.png)

![image](https://user-images.githubusercontent.com/39546083/122408096-126f8880-cfbd-11eb-8bec-c27927b54396.png)

VPN 안에 IPSec 을 만들어 주는 작업이다. 이때 risk 를 없애기 위해 AWS Direct Connect 을 연결하기도 한다.

## VPC 및 다른 AWS 서비스

### VPC 내의 AWS 서비스

EC2 가 아닌 다양한 AWS 서비스를 VPC 안에 위치시킬 수 있다. → RDS, Lambda, RedShift, Elastic Cache, Elastic Search 등

VPC 를 제대로 만들어야 서비스간에 원활한 연결을 할 수 있다.

![image](https://user-images.githubusercontent.com/39546083/122408617-7f831e00-cfbd-11eb-91bc-30d6fc65e84c.png)

![image](https://user-images.githubusercontent.com/39546083/122411566-d1c53e80-cfbf-11eb-9b91-a5255861fd1d.png)

- S3 의 경우 특정 VPC 내부에 존재시키는 서비스가 아니다.
- VPC 안의 EC2 가 S3 와 통신하려면 IGW 를 통해서 통신해야 한다.
- 하지만 가까운 곳인데 인터넷을 사용하는 것은 불합리해 보인다. → 보안 관점에서 S3 는 IP 가 고정되어 있지 않다. 아웃바운드를 고정하는 보안을 갖는 회사의 경우 anywhere 로 오픈하는 것이 굉장히 힘들 것이다.
- 이때 사용하는 서비스가 `VPC endpoints for S3` 가 있다. → private 하게 S3 로 접속할 수 있다.(S3 로 향하는 라우팅 테이블을 만드는 것이다) → IAM 을 이용해 S3 접속 컨트롤을 세세하게 할 수 있다.

![image](https://user-images.githubusercontent.com/39546083/122409307-00dab080-cfbe-11eb-9515-2db14aac7d01.png)

### VPC Flow Logs: VPC traffic metadata CloudWatch Logs

세부적으로 설정한 규칙에 맞게 트래픽이 잘 이동되고 있는지에 대한 로그를 확인할 수 있어야 한다. → 방화벽 로그를 확인해 분석해야 한다.

![image](https://user-images.githubusercontent.com/39546083/122409679-48f9d300-cfbe-11eb-851c-6fcf4b4443e4.png)

- 위 그림에서 53번 포트가 reject 되었다 → 53번 포트는 DNS 서버가 사용하는 포트(UDP port)다.

## VPC: your private network in AWS

- VPC 를 만든다.
- VPC 내부의 AZ 별로 Subnet 을 만든다.
- Security Group 을 만들어 방화벽 정책(Network ACL)을 만든다.
- 특정한 서브넷은 인터넷과 연결하거나 다른 VPC 와 연결할 수 있다.

![image](https://user-images.githubusercontent.com/39546083/122410093-98d89a00-cfbe-11eb-97f8-1d5d9b691ece.png)

![image](https://user-images.githubusercontent.com/39546083/122410206-abeb6a00-cfbe-11eb-873b-fd2b503c693a.png)
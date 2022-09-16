# EC2 (Elastic Compute Cloud)
> 아마존에서 가장 인기있는 서비스이며 어디에나 사용되고 있다. 이는 AWS에서 제공하는 `서비스형 인프라 스트럭쳐`이다.

- EC2는 아래 내용들을 포함하고 있다.
  - 가상 머신을 임대 (EC2 인스턴스)
  - 가상 드라이브 또는 EBS에 데이터 저장
  - ELB(Elastic Load Balancer)로 트래픽을 분산 처리
  - 또한, ASG(Auto Scaling Group)을 통해서 서비스를 확장할 수 있다.

<br>

- 내가 원하는 대로 가상 머신을 선택해서 AWS에서 빌릴 수 있다는 것이 핵심이다.
- 즉, EC2 사용자 데이터 스크립트를 사용해서 인스턴스를 **부트스트래핑**할 수 있다.

### 부트스트래핑이란 ?
- 머신이 작동될 때 명령을 시작하는 것을 의미한다.
- 스크립트는 시작할 때 한번 실행되고 다시 실행되지 않는다.
- `EC2 사용자 데이터`에는 매우 특정한 목적이 있는데 **부팅 작업을 자동화하기 때문에 부트스트래핑이라는 이름을 갖게 된다.**
- (참고) EC2 사용자 데이터 스크립트는 루트 계정에서 실행되어진다 따라서 모든 명령은 `sudo`가 붙는다.


### 사용자 데이터 스크립트
```bash
#!/bin/bash
# Use this for your user data (script from top to bottom)
# install httpd (Linux 2 version)
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello World from $(hostname -f)</h1>" > /var/www/html/index.html
```
- 이는 EC2 인스턴스에 웹 서버를 시작하고 파일을 쓸 것이다.

## EC2 인스턴스의 종류
> 예시로 수백개 중 간단하게 알아보자.

### 1. t2.micro (프리티어)
- 매우 간단하며 vCPU(코어수) 1개, 메모리 1GB
- 스토리지는 오직 EBS
- 네트워크 성능은 낮음과 중간 사이

### 2. t2.xlarge
- t2의 같은 제품군으로 vCPU 4개, 메모리 16GB
- 네트워크 성능은 중간

### 3. c5d,4xlarge
- vCPU 16, 32GB, 초당 최대 10Gbps


- 수 많은 종류가 있으므로 본인의 APP에 가장 적합한 인스턴스를 선택해서 주문형 클라우드를 사용할 수 있음.


## 인스턴스 공인 IP와 관련
- 인스턴스를 중지 후 재시작 한다면 사설 IP는 바뀌지 않지만, 공인(public IP)는 새로 할당을 받게 된다. 따라서 인스턴스 중지 이전의 공인 IP로 접근할 수 없다.
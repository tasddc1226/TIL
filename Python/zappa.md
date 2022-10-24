# Zappa

> zappa란 서버리스 오픈소스 관리 툴이다. 이 툴을 통해서 인프라 구성과 배포를 자동화할 수 있다.

## 서버리스 파이썬 웹서비스가 어떻게 운영되고 있는지 ?

- 우선 서버리스를 선택한 이유에 대해서 알아보자.
    1. 제한적인 개발 리소스
        - 새로운 인프라 관리 인력을 채용하기 보단 기존의 개발자가 최대한 인프라에 신경쓰지 않도록 서비스를 개발, 운영할 수 있는 환경이 필요

    2. API 호출 트래픽 패턴
        - 특정 시점에 API 부하가 발생하게 된다.
        - 피크 타임 때 동일한 서버 성능을 보장하기 위해서는 잠시 리소스를 스케일 아웃 또는 업해야 하는 상황이 발생하게 된다.
        - 이 시점이 시즌/비시즌 주말/평일에 따라 매우 불규칙적이다.

- 서버리스 환경에서는 이러한 트래픽 패턴에 대해 개발자가 고민할 필요가 없음.


## 서버리스 파이썬 웹서비스 배포하기\

- [Zappa](https://github.com/zappa/Zappa)
- AWS 서버리스 환경 구성 및 배포를 관리해주는 서버리스 파이썬 웹 서비스 관리 오픈소스이다.
- IAM Role, 코드 패키징, API Gateway, Lambda 세팅등을 자동으로 수행
- Python + Django + Docker + AWS 서버리스 환경 구성 지원


### AWS credential

- Zappa를 본격적으로 세팅하기 이전에 AWS credential 세팅이 필요하다. (더 자세한 내용은 공식문서)
- credential 세팅이 완료되고 자파 설치 후 기본 세팅

```bash
$(venv) > pip install zappa
$(venv) > zappa init
$(venv) > cat zappa_settings.json
```

```json
// zappa settings.json
{
    "dev" : {
        "aws_region" : "us-east-1", // 해당 stage가 배포될 리전
        "django_settings" : "zappa_example.settings", // Django settings 파일의 경로 명시
        "profile_name" : "default", // AWS CLI Configuration profile 명시 
        "project_name" : "zappa-example", // 람다 함수의 프로젝트 명으로 세팅
        "runtime" : "python3.8", // 람다에서 동작할 런타임 환경 명시 (docker 이미지로 배포하는 경우 불필요)
        "s3_bucket": "zappa-yyamukhwg", // 코드 패키징 파일을 업로드할 s3 버킷을 명시
        "events" : [{
            "function": "task_1",
            "expression": "cron(0/2 * * * ? *)"
        }, {
            "function": "task_2",
            "expression": "cron(* * * * ? *)"
        },]
    }
}
```

- `dev` 라는 스테이지에 대한 내용이다. 스테이지는 여러개를 만들 수 있다.
- `dev`, `staging`, `production`을 보통 만든다.
- 각 스테이지를 통해 배포도 또한 따로따로 진행할 수 있다.
- `"event"`라는 항목에는 여러 가지 python 함수들을 크론 표현식으로 등록해 주면 주기적인 cron job들도 실행할 수 있다.

### Deploy

```bash
$(venv) > zappa deploy dev
```

- 위의 명령어 과정을 간략하게 정리
  - 코드를 zip 파일로 압축
  - s3 버킷에 업로드
  - 람다 함수, API Gateway 세팅
  - 배포가 성공하면 마지막에 출력되는 url을 통해 어드민 페이지 접속 가능

### Undeploy

```bash
$(venv) > zappa undeploy dev
```

- 위의 명령어로 AWS 리소스들을 한번에 회수할 수 있다.
  

### 추가로 가능한 기능들?

- 도커 이미지로 배포 기능
- 커스텀 도메인 및 SSL 설정
- **환경 변수 관리 기능**
- **비동기 태스크 구동 기능**
- **스케쥴러 관리 기능 등**


## Cold Start 지연 문제

> 자동차 내연기관 엔진을 추운곳에서 시동을 걸면 시동이 걸리는데 오래걸린다는 비유 반대 표현으로 Warm Start가 있다.

- AWS 람다에서 장시간 람다 함수를 사용하지 않다가 오랜만에 사용한다면 반응이 느리다.
- 왜냐하면 람다는 서버리스 형태이기 때문에 항시 가동중이 아님
- 그러므로 죽어있던 람다로 할당된 PC가 잠시 살아나는 과정이 필요하다.
- 해당 람다는 `메모리 슬립 상태`로 꺼진게 아니기 때문에 람다가 할당된 PC 부팅부터 람다에 포함시켜놓은 라이브러리 실행 등 언어마다 시간이 필요.

### cold start를 줄이려면..

- 지속적인 호출을 해주어야 한다?
- Event Bridge를 람다 함수에 걸어주어 일정 시점에 주기적으로 호출되어지도록 함.

> 하지만 zappa에서는 overhead를 예방하기 위해 약 4~5분? 간격마다 lambda를 짧게 호출해주는 warming 작업을 자동으로 수행해줍니다.


---

## 본격 zappa를 사용하기 위한 준비 과정

- 배포는 github action에서 이루어지는 것을 전제합니다.

### 1. 가상 환경 설정 (virtualenv 활용)

### 2. requirements.txt 만들기

### 3. AWS IAM 만들기, key 발급받기


### 4. Github action workflow 구성하기

### 5. VPC 구성

- lambda는 기본적으로 http 호출이 불가능하여 외부 환경과 격리되어 있다.
- 작동시키기 위해서는 트리거(Event Bridge)를 달아 실행할 수 있다.

#### VPC vs VPN

- VPC (Virtual Private Cloud)

  - AWS Cloud 내부에서 구성되는 사용자의 AWS 계정 전용 가상 네트워크
  - 이곳을 통해서 AWS 리소스를 시작할 수 있음.
  - AWS에서는 디폴트로 EC2-VPC를 제공한다.
  - **하지만**, Amazon VPC는 AWS의 확장 가능한 인프라를 사용한다는 이점과 함께 고객의 자체 데이터 센터에서 운영하는 기존 네트워크와 매우 유사합니다.
  - VPC는 Amazon 콘솔에서 생성된다.
  - 하나의 VPC는 하나의 Region 내에서만 생성 가능하지만 두 개 이상의 Region에 걸치는 것은 불가능하다.
  - **하지만**, 하나의 VPC는 여러개의 AZ(가용 영역, Availability Zone)에 걸쳐 생성될 수 있다.

- VPN (Virtual Private Network)

  - 인터넷을 통해 디바이스 간에 사설 네트워크 연결을 생성한다.
  - Public 네트워크를 통해 데이터를 안전하게 익명으로 전송하는데 사용한다.
  - 사용자 IP address를 마스킹하고 데이터를 암호화하여 수신 권한이 없는 사람이 읽을 수 없도록 한다.

### 6. lambda Security Group 만들기

### 7. NAT Gateway 설정(선택)

### 8. RDS 인스턴스 생성

### 9. DB setting

### 10. 환경 변수 설정

### 11. S3 버킷 생성

### 12. Zappa Settings.json

```json
{
    "dev": {
        "aws_region": "ap-northeast-2",
        "django_settings": "config.settings",
        "project_name": "opengallery",
        "runtime": "python3.8",
        "s3_bucket": "open-gellery-2022",
        "remote_env": "s3://open-gellery-2022/secret.json",
        "vpc_config":{
            "SubnetIds":["subnet-0d061a76af7334da7", "subnet-0ca66aece924a175e"],
            "SecurityGroupIds":["sgr-042fc4ab80bb5cff0", "sgr-00cb2f5b8b8b26fbc"]
        }
    }
}
```

### 13. Repository 업로드

- 배포할 프로젝트 레포지토리에 Github Action을 수행할 workflow 파일과 위에서 생성한 `zappa_settings.json`을 업로드
- 공개되어서는 안되는 정보 확인


### 14. workflow 실행

---

## 트러블 슈팅 정리

### 첫 git Action 실행 결과

<img width="435" alt="Screen Shot 2022-10-22 at 21 39 12" src="https://user-images.githubusercontent.com/55699007/197339620-082e13ca-293f-483b-ab2d-f7de718a8c2e.png">

> `Deploy with zappa` 부분 제외 모두 성공

- 오류 내용

<img width="1190" alt="Screen Shot 2022-10-22 at 21 44 03" src="https://user-images.githubusercontent.com/55699007/197339662-99304d7a-1346-41a4-b39a-5166cc1d6d5c.png">

> `SECRET_KEY`를 가져오지 못하고 있는 현상.

- .env 관리 방법?

  - [zappa](https://github.com/zappa/Zappa#remote-environment-variables-via-an-s3-file) 에서는 S3 버킷에 민감한 정보들을 업로드 하고 사용하는 것을 권장한다고 함.

  - AWS S3 BUCKET에 등록하여 가져오는 방법으로 하는게 좋은 듯?
  - *연결해야 할 RDS의 DB config들을 등록해야하는데.. 어떤 정보들을 사용해야하는지 모름.*

#### S3 버킷에 django SECRET_KEY를 등록하고 실행 결과

<img width="1206" alt="Screen Shot 2022-10-22 at 21 39 29" src="https://user-images.githubusercontent.com/55699007/197339667-c2c8851f-9878-4de2-99d1-83fdce71f75d.png">

> `SECRET_KEY`를 정상적으로 가져오는 것 확인.


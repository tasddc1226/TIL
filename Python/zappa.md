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


## 서버리스 파이썬 웹서비스 배포하기
- [Zappa](https://github.com/zappa/Zappa)
- AWS 서버리스 환경 구성 및 배포를 관리해주는 서버리스 파이썬 웹 서비스 관리 오픈소스이다.
- IAM Role, 코드 패키징, API Gateway, Lambda 세팅등을 자동으로 수행
- Python + Django + Docker + AWS 서버리스 환경 구성 지원


### AWS credential
- Zappa를 본격적으로 세팅하기 이전에 AWS credential 세팅이 필요하다. (더 자세한 내용은 공식문서)
- credential 세팅이 완료되고 자파 설치 후 기본 세팅

```bash
$ (venv) > pip install zappa
$ (venv) > zappa init
$ (venv) > cat zappa_settings.json
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
$ (venv) > zappa deploy dev
```
- 위의 명령어 과정을 간략하게 정리
  - 코드를 zip 파일로 압축
  - s3 버킷에 업로드
  - 람다 함수, API Gateway 세팅
  - 배포가 성공하면 마지막에 출력되는 url을 통해 어드민 페이지 접속 가능

### Undeploy

```bash
$ (venv) > zappa undeploy dev
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
# Locust
> 서버 부하 테스트를 해보자!

- `locust`는 설치와 사용이 편리하고, 테스트 시나리오를 파이썬 스크립트로 작성을 하기 때문에 다양한 시나리오 구현 가능!

<br>

## 모듈 설치
> pip install locust

## 테스트 스크립트 작성
> 아무 곳에서나 `locustfile.py`를 생성하고 아래와 같이 타이핑

```py
"""locust를 사용해서 시킬 작업들을 작성"""

from locust import HttpUser, task, between


class WebsiteTestUser(HttpUser):
    """
    부하테스트를 하면서 사용하게 될 유저의 객체를 생성하는데 사용
    HttpUser class를 상속
    HttpUser class는 각 유저에게 client 속성 부여
    그러므로 실제 User가 사용하듯 http request 요청을 전송
    """

    """
        최소 1초 ~ 최대 2.5초까지 대기 후 재요청
        즉, 유저별로 task 하나가 끝날 때마다 1 ~ 2.5초의 랜덤 대기시간
    """
    wait_time = between(1, 2.5)

    # 수행할 task를 정의
    # task가 여러개라면 가중치를 주어 실행 가능
    @task
    def get_all_users(self):
        # 테스트를 원하는 엔드포인트 작성
        self.client.get("/users/")

    @task(5)
    def get_one_user(self):
        self.client.get("/users/1")

```

## 스크립트 실행
> locust -f locustfile.py 명령으로 실행 후, http://0.0.0.0:8089/ 로 접속


## 실행 화면

<img width="442" alt="Screen Shot 2022-09-13 at 6 42 51 PM" src="https://user-images.githubusercontent.com/55699007/189868936-80990a3e-b6d0-46b4-909f-31e661a72d88.png">


- 총 몇 명의 유저로 테스트 할 것인지, 초당 몇 명의 유저를 늘릴 것인지 입력 후 실행
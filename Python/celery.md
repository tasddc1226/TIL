# Celery

## 들어가기 전에..
> Celery 4.0 버전은 장고 1.8 이상의 최신 버전을 지원한다. 따라서 장고 1.8 미만이라면 Celery 3.1 버전을 사용!

> 또한 같이 사용하는 모듈인 `celery-singleton`과 `django-celery-beat`가 있다. 추가로 더 알아보자.

<br>

## 모듈 설명
- Python으로 작성된 분산 메시지 전달을 기반으로 한 `비동기 작업 큐(Asynchronous task queue/Job queue)`이다.
- 셀러리를 사용해 분산처리 프로세스를 작성할 수 있다.
- **비동기**로 작업을 처리할 수 있도록 도와주는 파이썬 프레임워크이다.
- 따라서 셀러리는 **worker(워커)** 라고 불리는 프레임워크이다.


<br>

## 사용하는 이유?
- 웹 서버는 동기적으로 처리하기 때문에 오래 걸리는 연산이나 오래 걸리는 작업의 경우 사용자는 웹 서버의 처리가 모두 마무리될 때까지 대기해야 한다.
- 따라서, 오래 걸리는 작업을 비동기 처리 방식을 사용해 사용자가 해당 작업을 기다리지 않고 다른 작업을 진행할 수 있도록 사용자 측면에서 속도 개선을 유도하기 위해 사용한다.

<br>

## Flow 비교 해보기
1. Celery를 사용하는 경우
  - 사용자가 수행 시간이 오래 걸리는 작업 요청
  - 서버는 해당 작업 처리 시작
  - 유저는 서버에서 결과를 받을 때까지 작업 시간만큼 대기

2. Celery를 사용하지 않는 경우
  - 사용자가 수행 시간이 오래 걸리는 작업 요청
  - 요청을 받은 `View`에서는 `Broker`에게 해당 작업 실행 위임하고 각 작업을 구분할 수 있는 `Task ID` 발급 받음
  - 위임된 작업은 `Broker`가 놀고있는 `Worker`에게 전달하여 `Worker`가 비동기로 수행

### 구체적인 예시
- 어떤 플랫폼에서 유저의 액션에 따라서 여러 API에 카카오 알림톡을 발송한다고 가정해보자.
- 이때 외부 API를 연동하여 플랫폼에서 사용자에게 메시지를 전달해주는 것이 아니라, 외부 API에서 사용자에게 전달해주기 때문에 플랫폼 서버에서는 알림톡 전송을 기다리지 않고 전송하는 로직을 `Worker`에게 위임하여 비동기로 처리한다면 API 응답 속도를 단축시킬 수 있다.

<br>

## Broker ?
<img width="682" alt="Screen Shot 2022-08-24 at 6 31 58 PM" src="https://user-images.githubusercontent.com/55699007/186384154-781ef33f-a3ee-469e-9ebe-edff92139e75.png">

- 셀러리는 작업을 브로커에게 전달하면 워커가 해당 작업을 처리하는 구조
- 한마디로 브로커는 요청한 작업들을 담아두는 **큐**라고 생각하면 이해하기 쉽다.

<br>

### **브로커와 백엔드에 대한 비유**
```
브로커와 백엔드가 헷갈리신다면 콜센터에 근무하는 직장인을 한번 떠올려 보세요.

출근해서 자리에 앉으니 쉴 틈 없이 전화가 계속 울립니다. 여기 저기서 고객들은 불만 사항들을 쏟아냅니다. 문의 하나 받고 전화선을 뽑아버립니다. 그리고 그 일을 처리한 후 다시 전화선을 연결합니다. 이런식으로 콜센터 직원이 일을 한다면 고객들은 화가 머리끝까지 날 겁니다. "전화는 도대체 왜 안받는거야?" "도대체 뭐하고 있는거야!"

그래서 콜센터 직원은 일하는 방식을 바꾸기로 합니다. 오는 전화는 계속 받되 고객들이 얘기한 불만을 칠판에 순서대로 적어두기로요. 그러면 각 부서 직원들이 칠판에 적힌 내용을 보고 처리하는 거죠. 이렇게 되면 콜센터 직원은 전화를 계속 받을 수 있게 됩니다. 고객들도 자기 불만이 접수되었으니 기다리기로 합니다. 여기서 '칠판' 역할을 하는 것이 '브로커'입니다.(정확히 말하면 메시지큐이고, 이걸 순서대로 처리하는 것이 브로커입니다.)

그럼 일이 어떻게 처리되었는지 체크해서 고객에게 다시 알려주어야 하는 경우도 있겠지요? 예를 들어, A제품이 재입고되면 알려주세요. 라는 고객 문의가 있었다면 '재입고 여부'를 알 수 있어야 고객에게 결과를 알려줄 수 있습니다. 이 때, 결과를 기록하는 곳이 '백엔드'입니다. 경우에 따라 그냥 고객이 불만만 이야기하고 끝내는 경우와 같이 결과가 필요없는 경우도 있습니다. 이런 전화만 온다면 '백엔드'가 필요 없을 수도 있습니다. 하지만, 물건의 입고 여부, 반품 회수일, 환불 가능 여부 등 '결과'가 필요한 경우엔 '백엔드'가 필요합니다. 각 부서에서는 문제를 처리하고 결과를 '백엔드'에 기록하거든요. 콜센터 직원은 '백엔드'를 보고 고객에게 알려주고요.

실제 경우를 예로 들면, 어떤 게시물에 사람들이 좋아요를 누르면 빨간 하트가 표시되는 기능이 있다고 가정해보겠습니다. 수 백명의 사람들이 좋아요를 누르고 각각의 사람들 피드에 좋아요 누른 게시물에 빨간 하트가 표시되야 합니다. 그런데 이걸 사람들에게 기다리도록 만든다면 이런 메시지를 보여줘야 할거에요. "홍길동 님이 누른 좋아요는 298명의 요청을 처리 후 가능합니다. 잠시만 기다려 주세요." 사람들은 화가 나서 이 기능을 쓰지 않겠죠. 이걸 셀러리를 이용해 처리한다면, 화면에는 즉시 빨간 하트를 표시해줍니다. 그리고 사람들이 누른 좋아요를 메시지큐에 담에 '브로커'가 처리할 수 있게 하고, 결과를 '백엔드'에서 받도록 하는 겁니다. 무엇을 '백엔드'와 '브로커'로 쓸지는 자기의 상황에 맞게 선택하면 됩니다. 
```

<br>

### Celery 아키텍쳐

<img width="708" alt="Screen Shot 2022-09-14 at 3 08 05 PM" src="https://user-images.githubusercontent.com/55699007/190073120-f9ecdbd8-0389-4f82-95ad-e472fde914c0.png">


- Django 서버에서 tasks를 `Message Broker`를 통해 전달하면 하나 이상의 `Celery`가 `Task Queue`에 있는 Task를 받아서 처리한다.
- 위의 사진에서는 `Message Broker`의 역할이 `redis`와 `RabbitMQ`이다.

#### Redis 참고
> 컴퓨터 메모리를 이용한 (in-memory) Cache Server로 `Key / Value`를 이용해 Celery가 처리할 작업을 보낸 후 Cache에서 해당 Key를 제거하는 방식으로 작동한다. **Redis**는 데이터 검색을 위해 DB에 접근하기 전 메모리에서 Cache를 가져다 쓴다는 점에서 속도가 빠른 장점이 있다.


## PyCon Korea Celery의 빛과 그림자
> 정민영님의 파이콘 2015 영상을 보고 정리한 내용.

### 비동기 처리기는 왜 필요한걸까?
- 비동기 처리기는 동기적으로 수행하지 않아도 되는 일들을 처리해주는 역할을 한다.
- 결과를 즉시 받을 필요가 없거나, 지연하여 처리해야 되는 일들을 보통 처리한다.
- 제대로 처리가 되지 않아도 된다는 얘기는 아니기 때문에 별도의 잘 만들어진 처리기가 필요하다.

### 사용하는 이유? (추가)
- 쉽게 연동 가능
- 당신이 상상할 수 있는 모든 기능 제공
- 가장 많이 사용되어 지고 있음.


### Broker
- Celery는 무수히 많은 Broker를 지원하지만, RabbitMQ를 제외한 Redis이하는 일부 기능이 제한되어진다.
  - 과연 지금도 그럴까,,?
  - Celery는 원래 RabbitMQ랑 사용하려고 만들어진 친구이다?
  - Redis를 Broker로 사용한다면 모니터링 기능을 사용할 수 있음.
  - 가능하면 RabbitMQ를 사용해라..?


### AMQP
> 참고: Celery가 연결을 시도하는 대상에서 URL과 유사한 구문을 볼 수 있습니다. 프로토콜 이름 은 Advanced Message Queuing Protocolamqp 의 약자이며 Celery가 사용하는 메시징 프로토콜입니다. AMQP를 기본적으로 구현하는 가장 잘 알려진 프로젝트는 RabbitMQ이지만 Redis도 프로토콜을 사용하여 통신할 수 있습니다.

- 단순하게 Celery만 설치하는 것으로는 충분하지 않다.
- 추가로 Broker가 필요하므로 Broker 없이 실행하면 아래와 같은 오류가 발생한다.

```bash
[2022-09-21 18:04:03,841: ERROR/MainProcess] consumer: Cannot connect to amqp://guest:**@127.0.0.1:5672//: [Errno 61] Connection refused.
Trying again in 2.00 seconds... (1/100)
```


![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FFaJc6%2FbtqDVu3tKWO%2FImf27X4sWOa4Kkxz8Lp9VK%2Fimg.png)


### Celery prefetch의 배신?
- 우리가 생각하는 prefetch에 대한 일반적인 기대로는 Task들을 그냥 미리 땡겨두는 것이다. 단순하게.
- 하지만 문서에는 다음과 같이 적혀있다고 한다.

```md
Prefetch is a term inherited from AMQP that is **often misunderstood** by users.
The prefetch limit is a limit for the number of tasks (messages) a worker can reserve for itself
```

[ S ][ S ][ S ][ L ][ S ][ S ][ S ][ S ]
- 큐 size가 8인 브로커에 메시지가 쌓여있는 모습
- prefetch의 크기가 4라고 가정해보자.

[ S ][ S ][ S ][ L ]
- 그러면 워커에는 메시지가 위와 같이 들어가게 된다.
- S는 수행 시간이 짧은 task이고, L은 수행 시간이 긴 task이다.

[ L ][ S ][ S ][ S ]
- S가 빠르게 처리된 이후의 워커 상황은 위와 같을 것.
- L 뒤에 S가 채워진 모습이 우리가 기대하는 prefetch의 모습이다.


- 하지만, 실제로는 그렇지 않다! prefetch된 단위 천체의 작업을 소비해야 (ack*) 다음 prefetch가 수행된다.
- task가 비워지는 대로 다음 task를 broker에서 가져올거라는 일반적인 기대와는 차이가 있다.**(often misunderstood)**


### 주의 사항
- Task를 한 큐에 담지 마세요.
  - prefetch의 특성상 평균 수행 시간이 비슷한 것들이 같은 큐에 있는 것이 성능상 훨씬 유리하다.
  - Task의 절대적인 수 자체도 중요한 요소이다.
  - 처리의 중요도 / 시급도(priority)에 따른 분류도 중요하다.

### 정말 간단한게 성능에 큰 영향을 주는 **또 다른**요소
- 셀러리 옵션에는 `ignore_result`라는 옵션이 있다.
  - 이는 default로 **꺼져** 있다.
  - Celery는 기본적으로 수행 결과(return)를 `저장`해야 작업이 끝난다.
  - 하지만 대부분 Task내에서 직접 결과를 다른 곳에 저장하지, `return` 자체를 쓰는 경우는 드물다!
  - 결과를 저장하는 비용이 적지 않기 때문에 이 옵션을 켜두기만 해도 성능이 좋아진다?
  - 하지만 해당 옵션은 `.apply_async()`에 해당한다.

- 자습서 상의 내용에서 발췌를 해보면

```py
send_feedback_email_task.apply_async(args=[
  self.cleaned_data["email"], self.cleaned_data["messages"]
  ]
)
```
- 위와 같이 `.apply_async`를 사용하면 다양한 실행 옵션(countdown, retry etc.)을 주어 셀러리를 사용할 수 있지만,
- `.delay()` Celery에게 task message를 가장 빠르게 보낼 수 있는 방법이라고 한다.
- 그냥 `.delay()`를 사용하자?

<br/>

# 이젠 진짜 사용해보자.
- [Asynchronous Tasks With Django and Celery](https://realpython.com/asynchronous-tasks-with-django-and-celery/#integrate-celery-with-django) 의 자습서를 보고 실습 부분만 정리
- 지금까지는 Celery의 이론 내용에 대해서만 파악해보았고, 실제로 비동기 처리를 Django 프로젝트에서 사용해보자


## 1. project clone
- 자습서 내에서 프로젝트를 다운받는 링크에서 메일 주소를 통해서 받을 수 있다.
- Django 프로젝트 설정하는 방법은 `PASS`

<img width="208" alt="Screen Shot 2022-09-22 at 11 29 59 AM" src="https://user-images.githubusercontent.com/55699007/191645233-660b2876-010c-47f5-9d80-528fd5dd6d5c.png">

- 압축을 풀면 폴더 2개가 있는데 `initial` 폴더에서 작업을 한 결과가 `final`에 들어있는 듯 하다.

- `initial` 앱을 실행하면 아래와 같은 화면이 나온다.

<img width="754" alt="Screen Shot 2022-09-22 at 11 29 02 AM" src="https://user-images.githubusercontent.com/55699007/191645223-264aa4ce-0e0e-4ebb-a415-8a7710f3b603.png">


## 2. 코드 살펴보기
- 우선 첫 화면에서 알 수 있듯이 피드백을 받는 양식이고, 메일 주소와 메시지를 제출하는 앱이다.
- `initial` 앱의 해당 양식을 채우고 전송을 누르면 빈 화면에 처리중인 로딩 모습만 나오게 된다.

```py
# feedback/form.py

def send_email(self):
  """Sends an email when the feedback form has been submitted."""
  sleep(20)  # Simulate expensive operation that freezes Django
  send_mail(
      "Your Feedback",
      f"\t{self.cleaned_data['message']}\n\nThank you!",
      "support@example.com",
      [self.cleaned_data["email"]],
      fail_silently=False,
  )
```
- 실제 메일을 보내는 기능 내에서 딜레이를 주었기 때문이다. 실제로는 이렇게 사용하진 않지만 실제로 약간의 딜레이가 존재한다.
- 그렇다면 이제 이 부분을 비동기로 처리하기 위해서 필요한 `broker`와 `celery` 설정을 해보자.

## 3. Celery 설정하기
- pip 명령을 통해 설치

```bash
> pip install celery
```

- 앞서 말했듯이 celery만 설치한다고 해서 바로 동작하지는 않는다. 진짜?
- 하기의 명령어를 입력해보자.

```bash
❯ python -m celery -A django_celery worker
```

- celery는 작업 대기열에 작업을 보내는 프로그램과 통신하기 위해 브로커가 필요하다. 따라서 브로커가 없으면 아래와 같이 계속해서 연결을 시도하는 에러가 출력되어 진다.

```bash
[2022-09-22 02:39:33,822: ERROR/MainProcess] consumer: Cannot connect to redis://localhost:6379//: Error 61 connecting to localhost:6379. Connection refused..
Trying again in 2.00 seconds... (1/100)
```

- 에러를 살펴보면, Celery가 연결 시도하려는 대상은 URL과 유사한 구문임을 알 수 있다. 위에서 설명한 AMQP를 사용하며 Celery가 사용하는 메시징 프로토콜이다.
- 가장 잘 알려진 메세징 프로토콜은 RabbitMQ이지만, Redis로도 구현이 가능하다.

## 4. Redis 설치하기
- homebrew가 설치되어있다고 가정하고 하기의 명령 입력

```bash
> brew install redis
```

- 설치가 끝나면 다음으로 바로 redis 서버를 실행시켜준다.

```bash
> redis-server
```

- 무언가 멋진 로고가 뜨고, Redis가 연결 대기중인 모습인 것 같다.

## 5. python 프로젝트에서 redis를 사용하기 위한 모듈 설치하기
- 다음으로는 Redis와 연결하기 위한 python 인터페이스를 설치하자.

```bash
> pip install redis
```

> 주의 사항
> - 시스템에 Redis를 설치하고, 가상 환경에 redis-py를 설치해야 Python 프로그램에서 Redis로 작업할 수 있다.


## 6. Django 프로젝트에 셀러리 추가하기
- 마지막으로는 Django App을 메시지 생성자로 작업 대기열에 연결하는 것이다.

### `celery.py` 생성

```bash
django_celery/
├── __init__.py
├── asgi.py
├── celery.py
├── settings.py
├── urls.py
└── wsgi.py
```

```py
# celery.py

import os
from celery import Celery

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "django_celery.settings")

# Celery 인스턴스 생성
app = Celery("django_celery")

# Celery 환경변수를 가져오기 위해서 해주는 설정으로 CELERY로 시작해야 한다는 뜻
app.config_from_object("django.conf:settings", namespace="CELERY")

# django 모든 app 하위 폴더에서 tasks.py 파일을 찾아 task들을 등록
app.autodiscover_tasks()

```

- 다음으로는 환경 변수를 세팅해주자.

```py

# django_celery/settings.py

# Celery settings
CELERY_BROKER_URL = "redis://localhost:6379"
CELERY_RESULT_BACKEND = "redis://localhost:6379"

```

- 위와 같이 설정해주면 Broker와 DB backend를 Redis로 사용한다는 의미이다.

- 마지막으로는 `__init__.py` 파일에 `celery_app을` 등록해주자

```py
# django_celery/__init__.py

from .celery import app as celery_app

__all__ = ("celery_app",)
```

- 위의 과정까지 마쳤다면, 이제 `@shared_task` 데코레이터를 사용할 수 있다!

## 7. 서비스들 실행해보자.
- 아직 비동기 처리할 작업을 등록하기 전에, 열심히 설정해왔던 서비스들을 실행해주자.

  1. 프로듀서: Django App
  2. 메시지 브로커: Redis Server
  3. 소비자: Celery App

- 각각 따로 터미널 창을 열어서 실행하자.
### Django App 개발 서버 실행
```bash
> python manage.py runserver
```

### Redis Server 실행
```bash
> redis-server
```

```
Opening port: bind: Address already in use
```
- 만약 이미 실행되어져 있다면 같은 포트로 실행할 수 없기에 재시작을 하거나 종류 후 실행해주자.

```bash
> redis-cli shutdown
```

### Celery App 실행
```bash
❯ python -m celery -A django_celery worker
```
- 실행할 때에는 `django_celery`와 같이 Celery App 인스턴스가 포함된 모듈의 이름을 제공해야 한다.


## 8. 비동기 처리를 위한 리펙터링
- 이제는 위에서 django 모든 app 하위 폴더에서 tasks.py 파일을 찾아 task들을 등록하는 함수
`app.autodiscover_tasks()`가 정상 동작하기 위해서 `tasks.py`를 작업해보자.

```bash
feedback/
│
├── migrations/
│   └── __init__.py
│
├── templates/
│   │
│   └── feedback/
│       ├── base.html
│       ├── feedback.html
│       └── success.html
│
├── __init__.py
├── admin.py
├── apps.py
├── forms.py
├── models.py
├── tasks.py
├── tests.py
├── urls.py
└── views.py
```

### tasks.py
```py
from time import sleep
from django.core.mail import send_mail
from celery import shared_task

@shared_task()
def send_feedback_email_task(email_address, message):
    """Sends an email when the feedback form has been submitted."""
    sleep(20)  # Simulate expensive operation(s) that freeze Django
    send_mail(
        "Your Feedback",
        f"\t{message}\n\nThank you!",
        "support@example.com",
        [email_address],
        fail_silently=False,
    )
```
- 비동기로 처리할 함수를 `tasks.py`에 등록했으니, 원래 동기적으로 실행하던 부분에서 호출만 하도록 아래와 같이 변경하자.

### forms.py
```py
from django import forms
from feedback.tasks import send_feedback_email_task

class FeedbackForm(forms.Form):
    email = forms.EmailField(label="Email Address")
    message = forms.CharField(
        label="Message", widget=forms.Textarea(attrs={"rows": 5})
    )

    def send_email(self):
        send_feedback_email_task.delay(
            self.cleaned_data["email"], self.cleaned_data["message"]
        )
```
- `tasks.py`에 등록된 함수를 가져오고, `.delay()`를 사용해서 호출해주는 모습이다.

## 9. Celery App 재실행

- 새롭게 `@shared_task`를 등록했기 때문에 이전에 실행해둔 Celery App을 아래의 명령어로 재시작 해주자.

```bash
❯ python -m celery -A django_celery worker -l info
```

- `-l info` 옵션을 주면 더 자세한 정보를 알 수 있다.
- 터미널에 어떤 `task`가 등록되었는지도 출력되어진다.

```bash
[tasks]
  . feedback.tasks.send_feedback_email_task
```

## 10. 마무리 실행 결과 
- 정말 비동기로 처리되어지는지 Celery App의 터미널을 주목하자.
- 아까와 같은 시나리오처럼 `email`과 `message`를 입력하고 제출을 하면 바로 결과 `html`을 받을 수 있다.
- 그리고 Celery App의 터미널을 확인하면 아래와 같다.

```bash
[2022-09-22 03:18:39,698: INFO/MainProcess] Task feedback.tasks.send_feedback_email_task[786f7992-53ea-4644-91a9-bfbbb3a733f6] received
[2022-09-22 03:18:59,724: WARNING/ForkPoolWorker-8] Content-Type: text/plain; charset="utf-8"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Subject: Your Feedback
From: support@example.com
To: 123@asd.asd
Date: Thu, 22 Sep 2022 03:18:59 -0000
Message-ID:
 <166381673970.31472.12423380710324090900@yangsuyeongs-MacBook-Pro.local>

	123

Thank you!
[2022-09-22 03:18:59,725: WARNING/ForkPoolWorker-8] -------------------------------------------------------------------------------
[2022-09-22 03:18:59,729: INFO/ForkPoolWorker-8] Task feedback.tasks.send_feedback_email_task[786f7992-53ea-4644-91a9-bfbbb3a733f6] succeeded in 20.030639417000003s: None
```
- `send_feedback_email_task`가 메시지를 수신함과 동시에 ID값이 존재하고, 이후에는 처리 결과에 대한 내용도 출력되어지는 것을 알 수 있다.
- 실행 시간도 확인할 수 있는 모습이다.
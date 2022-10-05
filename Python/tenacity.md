# Tenacity
> Tenacity는 Apache 2.0 라이센스의 파이썬으로 작성된 일반적인 목적의 **retrying 라이브러리**이다. 쉽게 task들의 재시도 행동을 추가할 수 있다.
> 이는 API에 대한 재시도와 호환되지 않지만 새로운 기능과 버그를 수정했습니다.

- 가장 간단한 사용 사례는 값이 반환될 때까지 예외가 발생할 때마다 비정상적인 함수를 재시도하는 것이다.


## Features
- Generic Decorator API
- 특정 정지 조건 (limit by number of attemps)
- 특정 대기 조건
- 예외에 대한 retry 커스터마이즈
- 예상되는 결과에 대한 커스터마이즈
- 코루틴에서 재시도
- context 관리자로 코드 블럭을 재시도


## Installation
> $ pip install tenacity


## 코드 블럭 재시도
- `Tenacity`를 사용하면 코드 블록을 격리된 함수로 래핑할 필요 없이 코드 블록을 다시 시도할 수 있다.
- 이렇게 하면 컨텍스트를 공유하는 동안 실패한 블록을 쉽게 분리가 가능하다.
- 방법은 for loop와 context manager를 결합하는 것이다.

```py
from tenacity import Retrying, RetryError, stop_after_attempt

try:
    for attempt in Retrying(stop=stop_after_attempt(3)):
        with attempt:
            raise Exception("My code is failing!")
    except RetryError:
        pass
```

## AsyncRetrying
- 아래와 같이 `AsyncRetrying`를 사용하고, `async`와 함께 비동기로도 처리할 수 있다.

```py
   from tenacity import AsyncRetrying, RetryError, stop_after_attempt

   async def function():
      try:
          async for attempt in AsyncRetrying(stop=stop_after_attempt(3)):
              with attempt:
                  raise Exception('My code is failing!')
      except RetryError:
          pass
```
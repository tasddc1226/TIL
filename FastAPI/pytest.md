# pytest를 사용해서 간단하게 테스트를 해보자.

> 사전 준비사항으로 작성된 fast api가 있어야 한다.
> `pip install pytest`를 통해서 pytest module을 설치한다.
> 비동기에 대한 테스트를 위해서는 추가로 `pytest-asyncio`까지 설치해주자.


## For Test

> 아래와 같이 새로운 유저를 생성하는 api가 있다. 테스트를 해보자!

```py
@router.post("/", status_code=status.HTTP_201_CREATED, response_model=schemas.UserResponse)
async def create_user(user: schemas.User, db: Session = Depends(Base.get_db)):
    """Create a new user"""
    hashed_password = hash_password(user.password)
    user.password = hashed_password
    try:
        new_user = models.User(**user.dict())
        db.add(new_user)
        db.commit()
        db.refresh(new_user)
    except IntegrityError:
        raise HTTPException(status_code=status.HTTP_400_BAD_REQUEST, detail="Duplicate email")

    return new_user
```

### 디렉토리 구조

- fast api의 구조는 이러하다.

```bash
.
├── README.md
├── app
│   ├── core
│   │   └── config.py
│   ├── database.py
│   ├── main.py
│   ├── models.py
│   ├── routers
│   │   └── user.py
│   ├── schemas.py
│   └── utils
│       └── password.py
└── tests
    └── test_users.py

6 directories, 18 files
```

- 우리가 테스트하려고 하는 API는 `app/routers/user.py`에 존재한다.
- 테스트코드는 `app` 디렉터리와 같은 위치에 있는 `tests` 라는 디렉터리 내에 있다.

### 본격적으로 테스트 코드를 작성해보자.

- 작성 방법

> 우선, FastAPI에서 제공하는 TestClient 객체를 사용하여 작성한 API 코드를 client 변수를 통해 요청하고 응답 상태 코드와 json 결과값을 테스트할 수 있다.

```py
# test_users.py

from app import schemas
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)


def test_새로운_유저_생성():
    res = client.post("/users/", json={"email": "test@example.com", "password": "test"})
    new_user = schemas.UserResponse(**res.json())

    assert res.status_code == 201
    assert new_user.email == "test@example.com"

```

- 테스트 실행 방법

> 테스트 실행은 `pytest tests/test_users.py` 과 같이 할 수 있으며, 실행 결과를 더 자세히 보려면 `-v` 옵션을 추가하여 아래와 같은 결과를 얻을 수 있다.

```bash
❯ pytest tests/test_users.py -v
=============================================== test session starts ================================================
platform darwin -- Python 3.9.13, pytest-7.1.2, pluggy-1.0.0 -- /Users/yangsuyoung/dev/fastapi/venv/bin/python3.9
cachedir: .pytest_cache
rootdir: /Users/yangsuyoung/dev/fastapi/blog
collected 1 item                                                                                                   

tests/test_users.py::test_새로운_유저_생성 PASSED                                                            [100%]

================================================ 1 passed in 0.50s =================================================
```

### 보완해야 할 점

- 동일한 테스트 코드로 같은 다시 한 번 테스트를 해보면, 아래와 같이 통과되어지지 않는다.

```bash
collected 1 item                                                                                                   

tests/test_users.py F                                                                                        [100%]

===================================================== FAILURES =====================================================
__________________________________________________ test_새로운_유저_생성 __________________________________________________

    def test_새로운_유저_생성():
        res = client.post("/users/", json={"email": "test@example.com", "password": "test"})
>       new_user = schemas.UserResponse(**res.json())

tests/test_users.py:12: 
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

>   ???
E   pydantic.error_wrappers.ValidationError: 3 validation errors for UserResponse
E   id
E     field required (type=value_error.missing)
E   email
E     field required (type=value_error.missing)
E   created_at
E     field required (type=value_error.missing)

pydantic/main.py:341: ValidationError
============================================= short test summary info ==============================================
FAILED tests/test_users.py::test_새로운_유저_생성 - pydantic.error_wrappers.ValidationError: 3 validation errors ...
================================================ 1 failed in 0.59s =================================================
```

- 그 이유로는 혹시나 API와 연결된 DB를 확인해보니.. 테스트코드에 담긴 계정과 비밀번호가 저장되어진 것이다.
- 따라서, 테스트용 DB를 config 해주지 않았기 때문이다. 해결하기 위해서는 테스트 코드가 실행되어질 때, 테스트 DB를 따로 config 해주거나 아니면 종료 후 만들어진 데이터를 일괄적으로 삭제하는 방법이 존재할 것 같다.

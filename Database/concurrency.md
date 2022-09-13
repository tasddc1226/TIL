# DB Concurrency
> 데이터베이스 정합성에 대한 내용 정리

<br/>

## 데이터베이스의 동시성?
- DB에 다수의 사용자가 동시에 접근하는 일은 비일비재 하다.
- **동시성은 이러한 상황에서 Transaction이 순차적으로 실행되는 것이 아니라, 순서에 상관없이 동시에 실행되는 것**을 의미한다.
- 따라서, 개발자는 동시성을 신경쓰고 관리해 줘야 한다.

<br/>

## Transaction 격리 수준
> RDBMS에서 처리하는 방식을 알아보자.

- 우선, Transaction 격리 수준이란 여러 Transaction이 동시에 처리될 때, 얼마나 고립되어 있는지를 나타낸다.

### RDBMS에서의 격리 수준
> RDBMS는 Transaction의 격리 수준을 제어할 수 있다.


- [DB Concurrency](#db-concurrency)
  - [데이터베이스의 동시성?](#데이터베이스의-동시성)
  - [Transaction 격리 수준](#transaction-격리-수준)
    - [RDBMS에서의 격리 수준](#rdbms에서의-격리-수준)
      - [READ UNCOMMITTED](#read-uncommitted)
      - [READ COMMITTED](#read-committed)
      - [REPEATABLE READ](#repeatable-read)
      - [SERIALIZABLE](#serializable)
  - [Django에서 select_for_update](#django에서-select_for_update)

#### READ UNCOMMITTED
- 한 Transaction의 변경이 commit, rollback에 상관없이 다른 Transaction에 영향을 미친다.
- 즉, T1(Transaction 1)이 데이터를 2에서 1로 수정(Update)한 후 commit하지 않았지만, T2는 Update 이후의 데이터인 1을 조회(READ)하게 된다.

> 발생하는 문제점

- `Dirty Read` 문제가 발생할 수 있다.
- 이 문제는 Transaction에 의해 삽입, 변경된 데이터를 commit 하기 전에 읽는 것을 의미한다.
- 만약 다른 Transaction이 rollback 된다면 다시 조회했을 때 데이터의 값이 달라지게 된다.
- 따라서 대부분의 RDBMS에서는 사용하지 않는다.

<br/>

#### READ COMMITTED
- 한 Transaction의 변경이 commit된 이후에만 변경사항이 다른 Transaction에 반영된다.
- T1이 값을 수정(2 -> 1)후, T2가 읽을 때 T1 commit 전에는 변경 전(2) 값을, T1 commit 이후에는 변경 후(1) 값을 조회한다.

> 발생하는 문제점

- `Non-Repeatable Read` 문제가 발생할 수 있다.
- 해당 문제는 한 Transaction이 같은 값을 조회할 때 다른 값이 조회되는 현상이다.
- 즉, T2가 조회하기 위해서 접근하는 값이 T1 commit 전, 후로 값이 다르다는 현상
- 해당 수준은 Oracle을 포함하여 많이 선택되는 격리 수준이다.

<br/>

#### REPEATABLE READ
- Transaction이 시작되고 종료되기 전까지 한 번 조회한 값은 같은 값으로 조회되도록 보장한다.
- update에 대해서는 정합성이 보장되지만, insert에 대해서는 보장되지 않는다.

> 발생하는 문제점

- `Phantom Read` 문제가 발생할 수 있다.
- Phantom, 즉 환상을 보는 것과 같이 데이터가 사라지거나 없던 데이터가 생겨나는 현상이다.
- 예를들어, T1이 ID가 2인 값을 조회에 성공했지만, T2가 rollback (2 -> 1) 한다면 ID가 2인 데이터는 없어진다.
- 다시 T2가 ID 2인 데이터를 조회하면 아무 데이터가 없다.

<br/>

#### SERIALIZABLE
- 가장 엄격한 격리 수준으로 한 Transaction이 테이블에 접근해 조회하고 있다면, 다른 Transaction은 그 테이블에 대하여 추가/변경/삭제가 불가능하다.
- 위의 언급한 세 개의 **정합성 문제는 발생하지 않지만 동시 처리 성능이 가장 떨어진다.**
- 따라서, Transaction의 격리 수준을 높인다면 동시성 문제를 해결할 수 있지만 성능은 반대로 대폭 하향될 것이다.

<br/>

## Django에서 select_for_update

```py
select_for_update(nowait=False, skip_locked=False, of=(), no_key=False)
```

> - nowait: 기본값은 False. False는 row lock이 잡혀있다면 대기하고 True일 경우 에러 발생
> - skip_locked: 기본값은 False. True일 경우 조회한 데이터가 lock이 잡혀있다면 이를 무시한다.
> - of: `select_for_update`는 `select_related`와 같이 사용할 경우 join 한 테이블의 행도 함께 lock을 잡는다. 이때 of를 사용해 lock을 잡을 테이블을 명시할 수 있다.



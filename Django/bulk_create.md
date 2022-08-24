# bulk_create
> **bulk_create**(objs, batch_size=None, ignore_conflicts=False, update_conflicts=False, update_fields=None, unique_fields=None)

<br>

### 함수 설명
- 비동기 버전으로 Django 4.1 버전에 abulk_create() 함수가 있다.
- 객체의 리스트를 insert 하는 메소드이다.
- 데이터베이스에 효율적으로 값을 생성한다.
- insert 하려는 객체의 수가 몇개나 있던 상관없이, bulk_create는 일반적으로 1개의 쿼리만 생성한다.
- 생성된 객체 리스트를 리턴한다.
- `batch_size`는 한 쿼리에 몇개의 객체를 생성할 것인지 제어하는 매개변수이다.
- `ignore_conflicts` 값은 `True`라고 한다면, 중복값과 제약조건에 걸려 행 insert에 실패하더라도 무시하도록 DB에게 지시하는 것이다. 또한, 모델의 각 인스턴스 기본키 설정이 비활성화 되어진다.

<br>

```bash
>>> objs = Entry.objects.bulk_create([
...     Entry(headline='This is a test'),
...     Entry(headline='This is only a test'),
... ])
```

<br>

### 주의 사항
- **bulk_create** 함수를 사용하면 모델의 **save()** 는 호출되어지지 않는다. pre_save()dhk post_save()에 대한 시그널도 발생하지 않는다.
- N : M(다대다) 관계에서는 동작하지 않는다.
- 다중 테이블 상속 시나리오의 자식 모델에서는 동작하지 않는다.
- 모델의 PK가 AutoField라면 DB 백엔드에서 지원하지 않는 한, **save()** 와 같이 기본키 설정을 하지 않는다.

<br>

### 비슷한 함수로 bulk_update()함수가 있다.
> **bulk_update**(objs, fields, batch_size=None)

- 또한 효율적으로 제공된 모델 인스턴스에 주어진 필드들을 업데이트 시켜준다.
- 일반적으로 하나의 쿼리만 사용되어진다. 그리고 업데이트된 개수를 리턴한다.

<br>

```bash
>>> objs = [
...    Entry.objects.create(headline='Entry 1'),
...    Entry.objects.create(headline='Entry 2'),
... ]
>>> objs[0].headline = 'This is entry 1'
>>> objs[1].headline = 'This is entry 2'
>>> Entry.objects.bulk_update(objs, ['headline'])
2
```

# iterator()
> **iterator**(chunk_size=None)


### 함수 설명
- django 4.1 버전부터 비동기 함수인 **aiterator()** 함수가 추가 되었다.
- 쿼리를 수행한 후 쿼리셋은 결과로 iterator를 반환한다.
- QuerySet은 일반적으로 평가를 반복하면 추가 쿼리가 발생하지 않도록 내부적으로 결과를 캐시한다.

- 이와 **대조적으로** 쿼리셋 레벨에서 어떤 캐싱 없이 iterator()는 직접적으로 결과를 읽는다.
- (내부적으로, 디폴트 iterator는 iterator()를 호출하고 리턴값을 캐시한다.)
- 한 번만 액세스하면 되는 많은 개체를 반환하는 QuerySet의 경우 성능이 향상되고 메모리가 크게 감소할 수 있다.

- *이미 평가된 QuerySet에서 iterator()를 사용하면 쿼리를 반복하면서 다시 평가해야한다.*

- iterator는 chunk_size가 주어지는 한 prefetch_related를 위한 이전 호출과 호환된다. 값이 클수록 메모리 사용량이 증가하여 프리페치를 수행하기 위해 필요한 쿼리가 줄어듭니다.

- [그러면 언제 iterator()를 사용하나요?](https://itecnote.com/tecnote/python-when-to-use-or-not-use-iterator-in-the-django-orm/)

### 추가 내용


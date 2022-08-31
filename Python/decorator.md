# decorator

> python decorator란 특정 함수를 wrapping 하고, wrapping 된 함수 앞뒤에 추가적으로 꾸며질 구문들을 정의하여 쉽게 재사용을 할 수 있도록 해준다.

### Nested function
```Py
def is_debug_enabled(func):
    def decorated():
        if DEBUG:
            # debug mode
            return func()
        else:
            return None

    return decorated
```
- 중첩된 함수 방식을 사용해서 데코레이트 함수를 정의할 수 있다.
- `@is_debug_enabled`라는 키워드를 통해서 사용할 수 있다.
- 해당 함수는 인자로 데코레이터가 적용된 함수를 인자로 받는다. Python은 함수의 인자로 또 다른 함수를 받을 수 있다는 특징을 사용한 것!
- 내부에 함수를 추가로 선언하여 실제로 작업할 내용을 선언할 수 있다. 위의 예시에서는 `def decorated()` 함수이다.
- 따라서 위의 데코레이터가 하는 일은, DEBUG mode라면 데코레이터가 붙은 함수를 실행하고 아니라면 실행하지 않게 하는 역할을 한다.

### Class 형태
```py
class DatetimeDecorator:
    def __init__(self, f):
        self.func = f

    # __call__ method로 decorated 정의
    def __call__(self, *args, **kwargs):
        print datetime.datetime.now()
        self.func(*args, **kwargs)
        print datetime.datetime.now()

class MainClass:
    @DatetimeDecorator
    def main_function_1():
            print "MAIN FUNCTION 1 START"
```
- 위와 같이 클래스 형태로도 데코레이터를 정의 및 사용할 수 있다.


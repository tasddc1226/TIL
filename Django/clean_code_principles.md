# Clean Code Principles in Django

> [장고에서 4가지 클린 코드 원칙들](https://betterprogramming.pub/clean-code-principles-in-django-b0563a4e12f5)을 읽고 번역 & 정리해보자

> 이 내용은 2022년 CSUI의 SW 엔지니어링 프로젝트 코스에서 다뤄진 개별 리뷰에 대한 내용이다.

<br>

## 소개
- 청소하는것은 아마 당신이 좋아하는 활동도 아닐 뿐만 아니라 확실하게 나의 일은 아니다.
- 내 방을 청소하거나 코드를 깨끗하게 하는 것은 아마 귀찮고 썩 내키지 않는 일일 수 있다.
- 하지만 그 둘은 같은 공통점을 가지고 있다. 그것은 바로 매우 중요하다는 것이다.

<br>

## 1. 클린 코드의 몇 가지 원칙들
- 이미 알고 있을 수도 있지만, 클린 코드는 컴퓨터 또는 컴파일러에 대한 것은 아니다. 바로 사람(개발자)을 위한 것이다.
- 컴퓨터들에게는 내가 어떻게 코드를 작성하건간에 상관이 없고, 단지 올바른 구문(Syntax)과 로직이 맞는지만 중요하다.
- 하지만, 개발자들은 컴퓨터가 아니다!

<br>

가장 잘 알려진 원칙으로는 다음과 같다:
  1. KISS (Keep It Simple (and) Stupid)
  2. DRY (Don't Repeat Yourself)
  3. YAGNI (You Aren't Gonna Need It)

<br>

각각의 원칙들을 실제 Django 코드를 통해서 알아보도록 하자.

<br>

## 2. KISS in Django
- 이 원칙은 기본적으로 다른 사람이 보아도 읽기 쉽게, 간단하게 그리고 짧게 유지하는 것이다.
- **Django Forms, Serializers, Permissions 그리고 Manual Alternatives 보다는 Decorators를 사용하는 것이 좋다.**
- Django를 배울 때, 아마도 Forms, Serializers 그리고 또 다른 것들을 많이 배웠을 것이다.

- Forms를 사용하지 않는 예시를 살펴보자.
```Python
# views.py

from django.shortcuts import render, redirect
from app.models import Object

def object_create(request):
    if request.POST:
        
        if request.POST.get("object_name") is None:
            # Handle bad data
            
        if request.POST.get("about_object") is None:
            # Handle bad data
        
	    object = Object.objects.create(
                object_name = request.POST.get("object_name"),
                about_object = request.POST.get("about_object"),
            )

	    return redirect("object:object_create_success", object.id)

	return render(request, 'objects/object_create.html', {
 		'form' : form,
	})
```

- 반대로 Forms를 사용하는 예시이다.

```Python
# forms.py

from django import forms
from app.models import Object

class ObjectCreationForm(forms.Form):
    object_name = forms.CharField(
        widget=forms.TextInput(attrs={'type' : 'text', 'placeholder' : 'Name of Object'})
    )
    about_object = forms.CharField(
        widget=forms.TextInput(attrs={'type' : 'text', 'placeholder' : 'About Object'})
    )

    def save(self, user):
        data = self.cleaned_data
        
        return Object.objects.create(
            object_name = data["object_name"],
            about_object = data["about_object"],
        )
```

```py
# views.py

from django.shortcuts import render, redirect
from app.forms import ObjectCreationForm

def object_create(request):
    form = ObjectCreationForm(request.POST or None)

    if request.POST and form.is_valid():
        object = form.save(request.user)
        return redirect("object:object_create_success", object.id)

    return render(request, 'objects/object_create.html', {
 	'form' : form,
    })
```

- 우선, Django Forms를 사용하는 코드가 두 부분으로 나눠지고, 사용하기 전 보다 더 긴 모습이다.
- 하지만 views.py 놓고 보았을 때 Form을 사용한 부분에서 훨씬 더 읽기 쉽고 간단해 보인다.
- 짧고 간결한 코드는 무엇을 하는지 쉽게 추론이 가능하다.
- 이것이 KISS 원칙의 중요한 포인트이다. 다른사람에게 읽기 쉽고, 간단한 코드를 만드는 것이 필요하다.

> The most important thing to take away from this is make simple and readable code for others.


### FBV보다는 CBV를 사용해라!
> 이전에 장고 프로젝트를 여러개 진행해보면서 CBV만 해보다가 설문조사 프로젝트를 진행하면서 FBV를 사용해보았다.
> 확실히 사용하면서 그 둘의 차이점을 확실하게 알 수 있었고, 장단점도 알 수 있었다.

- 필요한 경우가 아니면 FBV대신에 CBV를 사용해라. 그 이유는 특정 부분에서는 FBV가 더 유연할 지라도 CBV는 더 읽기 쉽기 때문이다.
- 더 자세한 내용으로는 [여기](https://betterprogramming.pub/refactors-you-need-to-know-to-for-your-django-project-8a56b0dee34f)를 보도록 하자.

<br>

## 3. DRY in Django
- Don't Repeat Yourself라는 뜻으로, 나의 코드에서 반복하지 않아야 한다는 의미이다.
- Django에서 중복성을 줄이기 위해 아래와 같은 작업들을 할 수 있다.

### Decorators를 사용해서 Views에 대한 엑세스 제한하기
- 데코레이터를 잘 모르고 있었다고 가정해보면 아마 아래와 같은 if문을 통해서 접근을 제한했을 것이다.

```Py
if request.user.is_authenticated:
```

- 그리고 여러개의 views.py가 있다면 아래와 같이 같은 일을 계속해서 중복할 것이다.

```py
from django.http import HTTPResponseRedirect
from django.urls import reverse

def a_view(request):
    if request.is_authenticated():
        # Do Someting
    else:
        return HttpResponseRedirect(reverse('users:login'))

def b_view(request):
    if request.is_authenticated():
        # Do Someting
    else:
        return HttpResponseRedirect(reverse('users:login'))
```

- 복사 붙여넣기가 아니라, 아래의 데코레이터를 활용한 방법을 살펴보자.

```Py
# decorators.py

def login_required(view_func):
    def wrapper_func(request, *args, **kwargs):
        if request.user.is_authenticated:
            # 유저가 인증되었다면, view의 함수에 접근할 수 있도록 함.
            return view_func(request, *args, **kwargs)
        else:
            # 유저가 인증되어지지 않았음. 그러므로 로그인 페이지로 이동시켜준다.
            return HttpResponseRedirect(reverse('user:login'))
    return wrapper_func
```

- 위처럼 데코레이터로 사용할 nested FUNCTION을 정의해주도록 하고 사용은 아래와 같이 해보자.

```Py
from app.decorators import login_required

@login_required
def a_view(request):
    #do something

@login_required
def b_view(request):
    #do something
```

- 커스텀하게 잘 사용할 수 있어 중복을 줄일 수 있다! The sky is the limit!

### 자주 사용하는 함수를 Helper 메소드로 따로 분리하여 사용해보도록 하자.
- 빈번하게 사용되어지는 코드를 분리하여 `utils.py`라는 파일에 분리를 해보자!
- 아래의 예시에서는 `helper function`이 없는 이메일을 보내주는 예제이다.

```py
from django.shortcuts import render
from django.core.mail import send_mail
from django.conf import settings

def send_to_requester_views(request):
    email_to_list = [request.user.mail]

    if settings.EMAIL_HOST_USER and settings.EMAIL_HOST_PASSWORD:
        send_mail(
            "Title of Email",
            "Descriptions",
            settings.EMAIL_HOST_USER,
            email_to_list,
        )
    return render(request. 'main/home.html')

def send_to_all_views(request):
    email_to_list = User.objects.value_list('email', flat=True)

    if settings.EMAIL_HOST_USER and settings.EMAIL_HOST_PASSWORD:
        send_mail(
            "Title of Email all User",
            "Descriptions",
            settings.EMAIL_HOST_USER,
            email_to_list,
        )
    return render(request. 'main/home.html')
```

- 위에서 반복되어지는 내용을 잘 살펴보자. 보이는 부분을 아래와 같이 새로운 파일 `utils.py`에 helper function으로 만들어주자.

```Py
from django.core.mail import send_mail
from django.conf import settings

def send_mail(email_subject, email_message, email_to_list):
    # 메일은 오직 호스트의 이메일과 비밀번호가 세팅되어 있어야 한다.
    if settings.EMAIL_HOST_USER and settings.EMAIL_HOST_PASSWORD and len(email_to_list) > 0:
        send_mail(
            email_subject,
            email_message,
            settings.EMAIL_HOST_USER,
            email_to_list,
        )
```

- 그러면 이제 `send_mail`을 호출하는 부분은 아래와 같이 수정되어질 수 있다.

```py
from django.shortcuts import render
from app.utils import send_mail

def send_to_requester_views(request):
    email_to_list = [request.user.email]
    send_mail("Subject of Email", "Hello!!", email_to_list)
    return render(request, 'main/home.home')

def send_to_all_views(request):
    email_to_list = User.objects.value_list('email', flat=True)
    send_email("Subject of Email for All", "Hello!!", email_to_list)
    return render(request, 'main/home.html')

```


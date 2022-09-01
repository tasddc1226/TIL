# Django viewsets

> 끝판왕 viewsets에 대해서 알아보자.

- 아래의 generics.ListAPIView를 사용한 코드이다.

```py
# views.py
from rest_framework import generics
from rest_framework.permissions import IsAuthenticated
from profiles.models import Profile
from profiles.api.serializers import ProfileSerializer

class ProfileList(generics.ListAPIView):
    queryset = Profile.objects.all()
    serializer_class = ProfileSerializer
    permission_classes = [IsAuthenticated]
```

- 위의 코드를 viewsets 형태로 바꾸어보자.

```py
# views.py
from rest_framework.permissions import IsAuthenticated
from rest_framework.viewsets import ReadOnlyModelViewSet
from profiles.models import Profile
from profiles.api.serializers import ProfileSerializer

class ProfileViewSet(ReadOnlyModelViewSet):
    queryset = Profile.objects.all()
    serializer_class = ProfileSerializer
    permission_classes = [IsAuthenticated]
```

```py
# urls.py
from django.urls import path
from profiles.api.views import ProfileViewSet

profile_list = ProfileViewSet.as_view({"get": "list"})
profile_detail = ProfileViewSet.as_view({"get": "retrieve"})

urlpatterns = [
    path("profiles/", profile_list, name="profile-list"),
    path("profiles/<int:pk>/", profile_detail, name="profile-detail")
]
```

- `urls.py`에서 특이하게 `as_view`의 인자로 딕셔너리를 인자로 전달하여 하나의 클래스인 `ProfileViewSet`에 대한 리스트와 디테일을 구분지어 연결시켜줄 수 있다.
- 이 동작이 가능한 이유로는 `ReadOnlyModelViewSet`의 내부를 확인해보면 다음과 같기 때문이다.

```py
# viewsets.py

class ReadOnlyModelViewSet(mixins.RetrieveModelMixin, mixins.ListModeMixin, GenericViewSet):
    """
    A viewset that provides default `list()` and `retrieve()` actions.
    """
    pass
```

- 한편, `ViewSetMixin` 클래스에서는 친절하게 사용방법도 알려준다.

```py
class ViewSetMixin(object):
    """
    This is magic.

    (생략))

    view = MyViewSet.as_view({'get': 'list', 'post': 'create'})
    """

    # (생략)
```

- 그런 다음, `DefaultRouter`를 적용해보자!

```Py
# urls.py

from django.urls import include, path
from rest_framework.routers improt DefaultRouter
from profiles.api.views import ProfileViewSet

# Before
# profile_list = ProfileViewSet.as_view({"get": "list"})
# profile_detail = ProfileViewSet.as_view({"get": "retrieve"})

# After
router = DefaultRouter()
router.register(r"profiles", ProfileViewSet)

# Before
# urlpatterns = [
#     path("profiles/", profile_list, name="profile-list"),
#     path("profiles/<int:pk>/", profile_detail, name="profile-detail")
# ]

# After
urlpatterns = [
    path("", inclue(router.urls))
]
```

- `DefaultRouter`를 사용하면 좀 더 간결하게 표현이 가능해 보인다.
- 동작은 `DefaultRouter`를 적용하기 전과 동일하지만, API 경로로 들어가보면 네이밍이 변경되어지고, `root`에 등록된 API들을 확인할 수 있는 `Api Root` 페이지가 추가로 생성되어 진다.



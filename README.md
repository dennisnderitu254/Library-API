# Library API Development using the django REST Framework

## Project Setup

Create a virtualenv `python3 -m venv env`

Add Django REST Framework using pip3

$ Shell
```
(env) > python -m pip install djangorestframework~=3.13.0
```

You have to notify django of the new installation in `django_project/settings.py`

in the `INSTALLED_APPS` section add `rest_framework`

`django_project/settings.py`
```
INSTALLED_APPS = [
"django.contrib.admin",
"django.contrib.auth",
"django.contrib.contenttypes",
"django.contrib.sessions",
"django.contrib.messages",
"django.contrib.staticfiles",
# 3rd party
"rest_framework", # new
# Local
"books.apps.BooksConfig",
]
```

Create an app for the project

$ Shell
```
(env) > python manage.py startapp apis
```

`django_project/settings.py`
```
INSTALLED_APPS = [
"django.contrib.admin",
"django.contrib.auth",
"django.contrib.contenttypes",
"django.contrib.sessions",
"django.contrib.messages",
"django.contrib.staticfiles",
# 3rd party
"rest_framework",
# Local
"books.apps.BooksConfig",
"apis.apps.ApisConfig", # new
]
```

Adding an API endpoint is just like configuring a traditional
Django URL route.

In the project-level `django_project/urls.py` file `include` the `apis app` and configure its URL route, which will be at `api/`

`django_project/urls.py`

```
from django.contrib import admin
from django.urls import path, include
urlpatterns = [
path("admin/", admin.site.urls),
path("api/", include("apis.urls")),
path("", include("books.urls")),
]
```

### Urls

Create a new file called `api/urls.py`

This file will import a future view called `BookAPIView` and set it to the URL route of "" so it will appear at `api/`

`apis/urls.py`

```
from django.urls import path
from .views import BookAPIView
urlpatterns = [
path("", BookAPIView.as_view(), name="book_list"),
]
```

### Views

Update the `apis/views.py`

```
from rest_framework import generics
from books.models import Book
from .serializers import BookSerializer

class BookAPIView(generics.ListAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
```


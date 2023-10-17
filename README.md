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

On the top lines, I imported Django REST Framework’s generics class of views, the `Book` model from our `books` app, and `serializers` from our `api` app. We will create the serializer used here, `BookSerializer` , in the following section.

Then I create a view class called `BookAPIView` that uses ListAPIView to create a read-only endpoint for all book instances.

The steps required in our view were to specify the queryset , which is all available books, and then the `serializer_class` which will be BookSerializer.

### Serializers

A `Serializer` translates complex data like querysets and model instances into a format that is easy to consume over the internet, typically JSON. It is also possible to “deserialize” data, literally the same process in reverse, whereby JSON data is first validated and then transformed into a dictionary.

The real beauty of Django REST Framework lies in its serializers which abstracts away most of the complexity for us. We will cover serialization and JSON in more depth in future chapters but for now the goal is to demonstrate how easy it is to create a serializer with Django REST Framework.

`apis/serializers.py`

```
from rest_framework import serializers
from books.models import Book

class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = ("title", "subtitle", "author", "isbn")
```

On the top lines we import Django REST Framework’s serializers class and the Book model from
our books app. Next, we extend Django REST Framework’s ModelSerializer into a `BookSerializer`
class that specifies our database model, Book , and the database fields we want to expose of `title` , `subtitle` , `author` , and `isbn`.

### Browsable API

Raw JSON data is not particularly friendly to consume with human eyes



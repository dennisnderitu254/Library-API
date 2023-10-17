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

Django REST Framework ships with a built-in browsable API that displays both the content and HTTP verbs associated with a given endpoint.

$ Shell
```
(.venv) > python manage.py runserver
```

- location of our API endpoint is at `http://127.0.0.1:8000/api/` so navigate there in your web browser.

**Add Image**

Django REST Framework provides this visualization by default. It displays the HTTP status code for the page, which is 200 meaning OK . Specifies Content-Type is JSON. And displays the information for our single book entry in a formatted manner.

**Add Image**

The Above data is not formatted at all and we can’t see any additional
information about HTTP status or allowable verbs either. I think we can agree the Django REST Framework version is more appealing.

Professional developers typically use on a third-party tool such as `Postman`  or, if on a Mac, Paw , to test and consume APIs. But for our purposes in this book the built-in browsable API is more than enough.

### Tests

Testing in Django relies upon Python’s built-in `unittest`, module and several helpful Django-specific extensions. Most notably, Django comes with a `test client` that we can use to simulate GET or POST requests, check the chain of redirects in a web request, and check that a given
Django template is being used and has the proper template context data.

Django REST Framework provides `several additional helper classes` that extend Django’s existing test framework. One of these is `APIClient` , an extension of Django’s default `Client` , which we will use to test retrieving API data from our database.

Since we already have tests in `books/tests.py` for our Book model we can focus on testing the API endpoint, specifically that it uses the URL we expect, has the correct status code of `200`, and contains the correct content.

Open the apis/tests.py file with your text editor and fill in the following code which we will review below.

```
https://www.postman.com/
https://paw.cloud/
https://docs.python.org/3/library/unittest.html#module-unittest
https://docs.djangoproject.com/en/4.0/topics/testing/tools/#the-test-client
https://www.django-rest-framework.org/api-guide/testing/
```

`api/tests.py`

```
from django.urls import reverse
from rest_framework import status
from rest_framework.test import APITestCase

from books.models import Book

class APITests(APITestCase):
    @classmethod
    def setUpTestData(cls):
        cls.book = Book.objects.create(
        title="Django for APIs",
        subtitle="Build web APIs with Python and Django",
        author="William S. Vincent",
        isbn="9781735467221",
        )

    def test_api_listview(self):
        response = self.client.get(reverse("book_list"))
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(Book.objects.count(), 1)
        self.assertContains(response, self.book)
```

At the top we import reverse from Django and from Django REST Framework both status and APITestCase . We also import our Book model though note that since we are in the api app we must specify the app name of book to import it.

We extend APITestCase in a new class called APITests that starts by configuring set up data. Then we run four different checks. First we check that the named URL of “book_list” is being used. Second we confirm that HTTP status code matches 200. Third we check that there is a single entry in the database. And finally we confirm that the response contains all the data from our created book object.

$ Shell
```
(env) > python manage.py test
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
...
----------------------------------------------------------------------
Ran 3 tests in 0.009s
OK
Destroying test database for alias 'default'...
```

Note that the output describes three tests passing because we had two in books/tests.py and one here. In larger websites with hundreds or even thousands of tests, performance can become an issue and sometimes you want to check just test within a given app before running the full
website test suite. To do that, simply add the name of the app you wish to check to the end of python manage.py test.

$ Shell
```
(env) > python manage.py test apis
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.005s
OK
Destroying test database for alias 'default'...
```

### Static Files

Static files are somewhat tricky to deploy properly on Django projects but the good news is that the process for Django APIs is essentially the same. Even though we do not have any of our own at this point, there are static files included in the Django admin and Django REST Framework browsable API so in order for those to deploy properly we must configure all static files.

First we need to create a dedicated static directory.

$ Shell
```
(env) > mkdir static
```

Git will not track empty directories so it’s important to add a `.keep` file so the static directory is included in source control.

Then we’ll install the `WhiteNoise` package since Django does not support serving static files in production itself.

$ Shell
```
(.venv) > python -m pip install whitenoise==6.0.0
```

WhiteNoise must be added to `django_project/settings.py` in the following locations:

• whitenoise above `django.contrib.staticfiles` in `INSTALLED_APPS`
• `WhiteNoiseMiddleware` above `CommonMiddleware`
• `STATICFILES_STORAGE` configuration pointing to WhiteNoise

In `django_project/settings.py`:

```
# django_project/settings.py
INSTALLED_APPS = [
    ...
    "whitenoise.runserver_nostatic",
    "django.contrib.staticfiles",
]
# new
MIDDLEWARE = [
    "django.middleware.security.SecurityMiddleware",
    "django.contrib.sessions.middleware.SessionMiddleware",
    "whitenoise.middleware.WhiteNoiseMiddleware", # new
    ...
]
STATIC_URL = "/static/"
STATICFILES_DIRS = [BASE_DIR / "static"] # new
STATIC_ROOT = BASE_DIR / "staticfiles" # new
STATICFILES_STORAGE =
"whitenoise.storage.CompressedManifestStaticFilesStorage" # new
```

The last step is to run the collectstatic command for the first time to compile all the static file directories and files into one self-contained unit suitable for deployment.

$ Shell
```
(.venv) > python manage.py collectstatic
```

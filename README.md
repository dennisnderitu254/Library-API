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

# Django 🐍

[![Django](https://img.shields.io/badge/Django-Web%20Framework-092E20)](https://www.djangoproject.com/)

> High-level Python web framework that encourages rapid development and clean design.

## Install
```bash
python -m venv .venv && source .venv/bin/activate  # Windows: .venv\\Scripts\\activate
pip install django djangorestframework psycopg2-binary python-dotenv
```

## Start Project
```bash
django-admin startproject config .
python manage.py startapp core
python manage.py migrate
python manage.py runserver
```

## Settings
```python
# config/settings.py
import os
from dotenv import load_dotenv
load_dotenv()

SECRET_KEY = os.getenv('SECRET_KEY', 'dev-secret')
DEBUG = os.getenv('DEBUG', 'true') == 'true'
ALLOWED_HOSTS = os.getenv('ALLOWED_HOSTS', 'localhost').split(',')

INSTALLED_APPS = [
    'django.contrib.admin', 'django.contrib.auth', 'django.contrib.contenttypes',
    'django.contrib.sessions', 'django.contrib.messages', 'django.contrib.staticfiles',
    'rest_framework', 'core',
]

DATABASES = {
  'default': {
    'ENGINE': 'django.db.backends.postgresql',
    'NAME': os.getenv('DB_NAME', 'app'),
    'USER': os.getenv('DB_USER', 'postgres'),
    'PASSWORD': os.getenv('DB_PASSWORD', 'postgres'),
    'HOST': os.getenv('DB_HOST', 'localhost'),
    'PORT': os.getenv('DB_PORT', '5432'),
  }
}

STATIC_URL = 'static/'
```

## Models
```python
# core/models.py
from django.db import models

class UserProfile(models.Model):
    user = models.OneToOneField('auth.User', on_delete=models.CASCADE)
    bio = models.TextField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
```

```bash
python manage.py makemigrations
python manage.py migrate
```

## Admin
```python
# core/admin.py
from django.contrib import admin
from .models import UserProfile
admin.site.register(UserProfile)
```

## Views & URLs
```python
# core/views.py
from django.http import JsonResponse

def health(_request):
    return JsonResponse({ 'ok': True })
```

```python
# config/urls.py
from django.contrib import admin
from django.urls import path, include
from core.views import health

urlpatterns = [
    path('admin/', admin.site.urls),
    path('health/', health),
]
```

## Templates
```html
<!-- core/templates/home.html -->
<!doctype html>
<html>
  <body>
    <h1>{{ title }}</h1>
  </body>
</html>
```

```python
# core/views.py
from django.shortcuts import render

def home(request):
    return render(request, 'home.html', { 'title': 'Welcome' })
```

## DRF API
```python
# core/serializers.py
from rest_framework import serializers
from .models import UserProfile

class UserProfileSerializer(serializers.ModelSerializer):
    class Meta:
        model = UserProfile
        fields = ['id', 'user', 'bio', 'created_at']
```

```python
# core/views.py
from rest_framework import viewsets, permissions
from .models import UserProfile
from .serializers import UserProfileSerializer

class UserProfileViewSet(viewsets.ModelViewSet):
    queryset = UserProfile.objects.all()
    serializer_class = UserProfileSerializer
    permission_classes = [permissions.IsAuthenticated]
```

```python
# core/urls.py
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import UserProfileViewSet

router = DefaultRouter()
router.register(r'profiles', UserProfileViewSet)

urlpatterns = [
    path('', include(router.urls)),
]
```

```python
# config/urls.py
urlpatterns = [
    path('admin/', admin.site.urls),
    path('health/', health),
    path('api/', include('core.urls')),
]
```

## Auth
```python
# config/settings.py
REST_FRAMEWORK = {
  'DEFAULT_AUTHENTICATION_CLASSES': [
    'rest_framework.authentication.SessionAuthentication',
    'rest_framework.authentication.BasicAuthentication',
  ],
  'DEFAULT_PERMISSION_CLASSES': [
    'rest_framework.permissions.IsAuthenticatedOrReadOnly',
  ]
}
```

## Testing
```bash
pip install pytest pytest-django
```

```ini
# pytest.ini
[pytest]
DJANGO_SETTINGS_MODULE = config.settings
python_files = tests.py test_*.py *_tests.py
```

```python
# core/tests/test_health.py
from django.urls import reverse

def test_health(client):
    resp = client.get('/health/')
    assert resp.status_code == 200
    assert resp.json()['ok'] is True
```

## Deployment
- Use `gunicorn` + `nginx`.
- Set `DEBUG=false`, secure cookies, `ALLOWED_HOSTS`.
- Run migrations on deploy.

## Resources
- Docs: djangoproject.com
- DRF: www.django-rest-framework.org


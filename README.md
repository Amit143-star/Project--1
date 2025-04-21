# Project--1
Open-Source Incident Management System


1. Project Overview

An incident management portal for logging, tracking, and resolving infrastructure or application issues with role-based access for Admins, Technicians, and Users.


2. Core Features

User Authentication & Role-Based Access

Roles: Admin, Technician, Reporter

JWT or Flask-Login/Django Auth for session handling


Incident Lifecycle Management

Create: Reporter logs an issue

Assign: Admin assigns it to a Technician

Update: Technician can update status/details

Resolve: Mark incident as resolved with resolution notes


Email Notifications (SMTP)

Triggered on: Create, Assign, Resolve

Config via .env file (SMTP credentials)


API Endpoints (Flask-RESTful or Django REST Framework)

POST /incidents/ - Create incident

GET /incidents/ - List all

PUT /incidents/<id> - Update/Assign/Resolve

GET /incidents/<id> - View details

Authenticated routes based on role


3. Tech Stack

Backend: Flask or Django

Database: SQLite

Frontend: Bootstrap + HTML templates (Jinja or Django templating)

Containerization: Docker (with Dockerfile + docker-compose.yml)

Version Control: Git + GitHub


4. Directory Structure (Flask Example)

incident-management/
├── app/
│   ├── __init__.py
│   ├── models.py
│   ├── routes.py
│   ├── email_utils.py
│   ├── templates/
│   └── static/
├── tests/
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── .env
└── README.md


5. Dockerization

Dockerfile (basic):

FROM python:3.11-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "run.py"]

docker-compose.yml:

version: '3.8'
services:
  web:
    build: .
    ports:
      - "5000:5000"
    env_file:
      - .env


6. Sample Data

Pre-fill the database with:

2-3 users per role

3-5 incident logs with various statuses


7. Deliverables

GitHub repo with source code

Docker image (on Docker Hub or exportable .tar)

Demo video/screenshots showing:

Login

Creating an issue

Assigning and resolving

Markdown file or DB JSON for sample issues

Full implementation plan for the Django-based Open-Source Incident Management System. I’ll split it into key components so we can build it step-by-step:


1. Project Setup

django-admin startproject incident_portal
cd incident_portal
python manage.py startapp incidents

Add 'incidents', 'rest_framework', and optionally 'corsheaders' to INSTALLED_APPS in settings.py.


2. Models (incidents/models.py)

from django.db import models
from django.contrib.auth.models import User

class Incident(models.Model):
    STATUS_CHOICES = [
        ('Open', 'Open'),
        ('Assigned', 'Assigned'),
        ('Resolved', 'Resolved'),
    ]

    title = models.CharField(max_length=200)
    description = models.TextField()
    reporter = models.ForeignKey(User, related_name='reported_incidents', on_delete=models.CASCADE)
    technician = models.ForeignKey(User, related_name='assigned_incidents', null=True, blank=True, on_delete=models.SET_NULL)
    status = models.CharField(max_length=10, choices=STATUS_CHOICES, default='Open')
    resolution_notes = models.TextField(blank=True, null=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)


3. Serializers (incidents/serializers.py)

from rest_framework import serializers
from .models import Incident
from django.contrib.auth.models import User

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'username', 'email']

class IncidentSerializer(serializers.ModelSerializer):
    reporter = UserSerializer(read_only=True)
    technician = UserSerializer(read_only=True)

    class Meta:
        model = Incident
        fields = '__all__'


4. Views (incidents/views.py)

from rest_framework import viewsets, permissions
from .models import Incident
from .serializers import IncidentSerializer
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from rest_framework.decorators import action
from django.core.mail import send_mail
from django.conf import settings

class IncidentViewSet(viewsets.ModelViewSet):
    queryset = Incident.objects.all().order_by('-created_at')
    serializer_class = IncidentSerializer
    permission_classes = [IsAuthenticated]

    def perform_create(self, serializer):
        incident = serializer.save(reporter=self.request.user)
        send_mail("New Incident Reported", f"{incident.title}", settings.DEFAULT_FROM_EMAIL, [incident.reporter.email])

    def perform_update(self, serializer):
        incident = serializer.save()
        if incident.status == 'Resolved':
            send_mail("Incident Resolved", f"{incident.title} has been resolved.", settings.DEFAULT_FROM_EMAIL, [incident.reporter.email])


5. URLs

In incident_portal/urls.py:

from django.contrib import admin
from django.urls import path, include
from rest_framework import routers
from incidents.views import IncidentViewSet

router = routers.DefaultRouter()
router.register(r'incidents', IncidentViewSet)

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include(router.urls)),
    path('api-auth/', include('rest_framework.urls')),
]


6. Authentication (Simple Login UI + API)

Use Django’s built-in login or add Token Authentication with djangorestframework.authtoken.


7. Email Config (settings.py)

EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_PORT = 587
EMAIL_USE_TLS = True
EMAIL_HOST_USER = 'your-email@gmail.com'
EMAIL_HOST_PASSWORD = 'your-password'
DEFAULT_FROM_EMAIL = EMAIL_HOST_USER


8. Docker Setup

Dockerfile

FROM python:3.11-slim
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["gunicorn", "incident_portal.wsgi:application", "--bind", "0.0.0.0:8000"]

docker-compose.yml

version: '3'
services:
  web:
    build: .
    ports:
      - "8000:8000"
    volumes:
      - .:/app
    env_file:
      - .env


9. Requirements

Django>=4.0
djangorestframework
gunicorn


10. Sample Users & Issues

Create a fixtures/initial_data.json for sample users and incident logs.


11. Optional

Frontend in Bootstrap for simple login/form views

JWT auth using djangorestframework-simplejwt

API documentation via Swagger or Postman collection


---
layout: post
title: Django custom users
date: 2024-10-16
categories:
- python
---

I've been looking at [Django](https://www.djangoproject.com/) recently and I wanted to share some notes on how to create custom users. Django comes with a built-in user model, but sometimes you need to extend it or create a new one. Here's how you can do it.

Basically there will be several steps to follow:
 - Create a custom user model
 - Update the myproject/settings.py file
 - Create a view for the custom user model
 - Update the admin confiruration
 - Create the migration and apply

## Optional: Create a new Django project

If you don't have a Django project yet, you can create one with the following command:

```bash
$ django-admin startproject myproject
```

```bash
$ cd myproject
```

## Create a custom user model

I went for the method of creating a new app in my project to manage users - called ```accounts```:

```bash
$ python manage.py startapp accounts
```


> **WARNING**: At this point, you may be tempted to create your superuser (```python manage.py createsuperuser```), but you'll get an error because Django doesn't know about your custom user model yet. Hold off that and create your custom user model first.

## Create a custom user model

To create a custom user model, you need to create a new model that inherits from `AbstractBaseUser` and `PermissionsMixin`. Here's an example:

```python
# accounts/models.py
from django.contrib.auth.models import AbstractBaseUser, BaseUserManager, PermissionsMixin
from django.db import models
from django.utils import timezone

# Create your models here.

class CustomUserManager(BaseUserManager):
    def create_user(self, email, password=None, **extra_fields):
        if not email:
            raise ValueError('The Email field must be set')
        email = self.normalize_email(email)
        user = self.model(email=email, **extra_fields)
        user.set_password(password)
        user.save(using=self._db)
        return user

    def create_superuser(self, email, password=None, **extra_fields):
        extra_fields.setdefault('is_staff', True)
        extra_fields.setdefault('is_superuser', True)

        if extra_fields.get('is_staff') is not True:
            raise ValueError('Superuser must have is_staff=True.')
        if extra_fields.get('is_superuser') is not True:
            raise ValueError('Superuser must have is_superuser=True.')

        return self.create_user(email, password, **extra_fields)

class CustomUser(AbstractBaseUser, PermissionsMixin):
    email = models.EmailField(unique=True)
    first_name = models.CharField(max_length=30, blank=True)
    last_name = models.CharField(max_length=30, blank=True)
    is_active = models.BooleanField(default=True)
    is_staff = models.BooleanField(default=False)
    date_joined = models.DateTimeField(default=timezone.now)
    objects = CustomUserManager()

    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = []

    def __str__(self):
        return self.email
```

Obviously, you can edit the model to suit all your needs (with extra fields, etc.).

## Update the myproject/settings.py file

Then I added the app to the `INSTALLED_APPS` list in the `myproject/settings.py` file:

```python
INSTALLED_APPS = [
    ...
    "accounts",
]
```

Ensure you have the following line in your `myproject/settings.py` file:

```python
# settings.py
...
AUTH_USER_MODEL = 'accounts.CustomUser'
```
## Create Views for the custom user model

```python
# accounts/views.py
from django.shortcuts import render

# Create your views here.

from django.shortcuts import render, redirect
from django.contrib.auth import login, authenticate
from .forms import CustomUserCreationForm, CustomAuthenticationForm

def register(request):
    if request.method == 'POST':
        form = CustomUserCreationForm(request.POST)
        if form.is_valid():
            user = form.save()
            login(request, user)
            return redirect('home')
    else:
        form = CustomUserCreationForm()
    return render(request, 'accounts/register.html', {'form': form})

def login_view(request):
    if request.method == 'POST':
        form = CustomAuthenticationForm(request, data=request.POST)
        if form.is_valid():
            user = authenticate(request, email=form.cleaned_data['username'], password=form.cleaned_data['password'])
            if user is not None:
                login(request, user)
                return redirect('home')
    else:
        form = CustomAuthenticationForm()
    return render(request, 'accounts/login.html', {'form': form})

from django.shortcuts import render

def home(request):
    return render(request, 'home.html')
```

## Update the admin configuration in accounts/admin.py and register the custom user model
```python
# accounts/admin.py

from django.contrib import admin
from django.contrib.auth.admin import UserAdmin
from .models import CustomUser

class CustomUserAdmin(UserAdmin):
    # Define the fields to be used in displaying the User model.
    list_display = ('email', 'first_name', 'last_name', 'is_staff', 'is_active')
    list_filter = ('is_staff', 'is_active')
    fieldsets = (
        (None, {'fields': ('email', 'password')}),
        ('Personal info', {'fields': ('first_name', 'last_name')}),
        ('Permissions', {'fields': ('is_staff', 'is_active', 'is_superuser', 'groups', 'user_permissions')}),
        ('Important dates', {'fields': ('last_login', 'date_joined')}),
    )
    add_fieldsets = (
        (None, {
            'classes': ('wide',),
            'fields': ('email', 'first_name', 'last_name', 'password1', 'password2', 'is_staff', 'is_active'),
        }),
    )
    search_fields = ('email', 'first_name', 'last_name')
    ordering = ('email',)

admin.site.register(CustomUser, CustomUserAdmin)
```

## Create the Migrations

Now we have the models, we need to create a migration to initialize the database. Run the following commands:

```bash
$ python manage.py makemigrations
$ python manage.py migrate
```

> **NOTE**: Now you can create the superuser. If you did this before, you will have an error.

```bash
$ python manage.py createsuperuser
```

Then you can run the development server and try and login:

```bash
$ python manage.py runserver
```

You can now go to the admin page and see your custom user model - at http://127.0.0.1:8000/admin/. 



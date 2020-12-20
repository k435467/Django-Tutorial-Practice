# Django-Tutorial-Practice

practice the Django tutorial on MDN web site. [link to the tutorial](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Django)

## Guides

- Django introduction
- Setting up a Django development environment
- Django Tutorial: The Local Library website
- Django Tutorial Part2: Creating a skeleton website
- Django Tutorial Part3: Using models
- Django Tutorial Part4: Django admin site
- Django Tutorial Part5: Creating our home page
- Django Tutorial Part6: Generic list and detail views
- Django Tutorial Part7: Sessions framework
- Django Tutorial Part8: User authentication adn permissions
- Django Tutorial Part9: Working with forms
- Django Tutorial Part10: Testing a Django web application
- Django Turorial Part11: Deploying Django to production
- Django web application security

## My notes

on Windows.

### Django introduction

![image of basic django](https://mdn.mozillademos.org/files/13931/basic-django.png)

### Setting up a Django development environment

- install Python, pip
- setup virtual environment
- install Django

### Django Tutorial: The Local Library website

[source on GitHub](https://github.com/mdn/django-locallibrary-tutorial)

### Django Tutorial Part2: Creating a skeleton website

- Creating the project

  - open a terminal, in the virtual environment (command: workon)
  - create a folder
  - create the new project

      ```shell
      django-admin startproject locallibrary
      cd locallibrary
      ```

- Creating the catalog application

  ```shell
  py manage.py startapp catalog
  ```

- Registering the catalog application
  - open the project settings file, and find the ```INSTALLED_APPS``` list

    ```python
    INSTALLED_APPS = [
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
        # Add our new application 
        'catalog.apps.CatalogConfig', #This object was created for us in /catalog/apps.py
    ]
    ```

- Specifying the database
  - open project settings file, and find ```DATABASES```

    ```python
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.sqlite3',
            'NAME': BASE_DIR / 'db.sqlite3',
            # 'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
        }
    }
    ```

- Hooking up the URL mapper
  - locallibrary/urls.py

    ```python
    from django.contrib import admin
    from django.urls import path
    from django.urls import include
    from django.views.generic import RedirectView
    from django.conf import settings
    from django.conf.urls.static import static

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('catalog/', include('catalog.urls')), # Use include() to add paths from the catalog application
        path('', RedirectView.as_view(url='catalog/', permanent=True)), # Add URL maps to redirect the base URL to our application
    ] + static(settings.STATIC_URL, document_root=settings.STATIC_ROOT) # Use static() to add url mapping to serve static files during development (only)
    ```

  - catalog/urls.py

    ```python
    from django.urls import path
    from . import views

    urlpatterns = [

    ]
    ```

- Run

  ```shell
  py manage.py makemigrations
  py manage.py migrate

  py manage.py runserver
  ```

### Django Tutorial Part3: Using models

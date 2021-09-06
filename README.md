# My notes

Practice the Django tutorial that is on the MDN website on Windows. [link to the tutorial](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Django)

Learn Django from scratch. Including setting up dev environment, defining models, Django admin site, auth and permissions, testing, security, etc. Build a local library website and deploy it on Heroku.

- [My notes](#my-notes)
  - [Django introduction](#django-introduction)
  - [Setting up a Django development environment](#setting-up-a-django-development-environment)
  - [Django Tutorial: The Local Library website](#django-tutorial-the-local-library-website)
  - [Django Tutorial Part2: Creating a skeleton website](#django-tutorial-part2-creating-a-skeleton-website)
  - [Django Tutorial Part3: Using models](#django-tutorial-part3-using-models)
  - [Django Tutorial Part4: Django admin site](#django-tutorial-part4-django-admin-site)
  - [Django Tutorial Part5: Creating our home page](#django-tutorial-part5-creating-our-home-page)
  - [Django Tutorial Part6: Generic list and detail views](#django-tutorial-part6-generic-list-and-detail-views)
  - [Django Tutorial Part7: Sessions framework](#django-tutorial-part7-sessions-framework)
  - [Django Tutorial Part8: User authentication and permissions](#django-tutorial-part8-user-authentication-and-permissions)
  - [Django Tutorial Part9: Working with forms](#django-tutorial-part9-working-with-forms)
  - [Django Tutorial Part10: Testing a Django web application](#django-tutorial-part10-testing-a-django-web-application)
  - [Django Turorial Part11: Deploying Django to production](#django-turorial-part11-deploying-django-to-production)
  - [Django web application security](#django-web-application-security)

## Django introduction

![image of basic django](https://mdn.mozillademos.org/files/13931/basic-django.png)

## Setting up a Django development environment

- install Python, pip
- setup virtual environment
- install Django

## Django Tutorial: The Local Library website

[source on GitHub](https://github.com/mdn/django-locallibrary-tutorial)

## Django Tutorial Part2: Creating a skeleton website

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

  - `settings.py`

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

  - `settings.py`

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

  - `locallibrary/urls.py`

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

  - `catalog/urls.py`

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

## Django Tutorial Part3: Using models

lookups files and docs.

```python
record = MyModelName(my_field_name="Instance #1") # Create using ctr.
record.save() # Save into database.
all_books = Book.objects.all() # get all records
wild_books = Book.objects.filter(title__contains='wild')
number_wild_books = wild_books.count()
```

## Django Tutorial Part4: Django admin site

- Registering models: `catalog/admin.py`

  ```python
  from .models import Author
  admin.site.register(Author)
  ```

- Creating a superuser

  ```shell
  py manage.py createsuperuser
  ```

- Register a ModelAdmin class: `catalog/admin.py`

  ```python
  @admin.register(Author)
  class AuthorAdmin(admin.ModelAdmin):
    list_display = ('last_name', 'first_name',
                    'date_of_birth', 'date_of_death') # list table cols
    fields = ['first_name', 'last_name', ('date_of_birth', 'date_of_death')] # detail view
    inlines = [BooksInline] # inline editing of associated records

  @admin.register(BookInstance)
  class BookInstanceAdmin(admin.ModelAdmin):
      list_filter = ('status', 'due_back') # filter box to the right

      # sectioning the detail view
      fieldsets = (
          (None, {
              'fields': ('book', 'imprint', 'id')
          }),
          ('Availability', {
              'fields': ('status', 'due_back')
          }),
      )
  ```

## Django Tutorial Part5: Creating our home page

- Referencing static files in templates

  ```html
  {% load static %}
  <link rel="stylesheet" href="{% static 'css/styles.css' %}" />
  <img src="{% static 'catalog/images/local_library_model_uml.png' %}" />
  ```

- Configuring where to find the templates: `settings.py`

  ```python
  TEMPLATES = [
      {
          'BACKEND': 'django.template.backends.django.DjangoTemplates',
          'DIRS': [], # specify locations to search
          'APP_DIRS': True,
          'OPTIONS': {
              # ...
              ],
          },
      },
  ]
  ```

## Django Tutorial Part6: Generic list and detail views

- Specify template, pass additional context variables. `views.py`

  ```python
  class BookListView(generic.ListView):
      # ...
      template_name = 'books/my_arbitrary_template_name_list.html'  # Specify your own template name/location

      # pass additional context variables
      def get_context_data(self, **kwargs):
          context = super(BookListView, self).get_context_data(**kwargs)
          context['some_data'] = 'This is just some data'
          return context
  ```

- Passing additional options in your URL maps

  ```python
  # use the same view for multiple templates
  path('url/', views.my_reused_view, {'my_template_name': 'some_path'}, name='aurl'),
  path('anotherurl/', views.my_reused_view, {'my_template_name': 'another_path'}, name='anotherurl'),
  ```

## Django Tutorial Part7: Sessions framework

- Enabling sessions: enabled automatically. `settings.py`

  ```python
  INSTALLED_APPS = [
      ...
      'django.contrib.sessions',
      ....

  MIDDLEWARE = [
      ...
      'django.contrib.sessions.middleware.SessionMiddleware',
      ....
  ```

- Saving session data

  ```python
  # Session object not directly modified, only data within the session. Session changes not saved!
  request.session['my_car']['wheels'] = 'alloy'

  # Set session as modified to force data updates/cookie to be saved.
  request.session.modified = True
  # or add project setting below
  # SESSION_SAVE_EVERY_REQUEST = True
  ```

## Django Tutorial Part8: User authentication and permissions

- Enabling authentication: enabled automatically. `settings.py`

  ```python
  INSTALLED_APPS = [
      ...
      'django.contrib.auth',  # Core authentication framework and its default models.
      'django.contrib.contenttypes',  # Django content type system (allows permissions to be associated with models).
      ...

  MIDDLEWARE = [
      ...
      'django.contrib.sessions.middleware.SessionMiddleware',  # Manages sessions across requests
      ...
      'django.contrib.auth.middleware.AuthenticationMiddleware',  # Associates users with requests using sessions.
      ...
  ```

- Project URLs: `locallibrary/urls.py`

  ```python
  urlpatterns += [
      path('accounts/', include('django.contrib.auth.urls')),
  ]
  # automatically maps the below URLs.
  # accounts/ login/ [name='login']
  # accounts/ logout/ [name='logout']
  # accounts/ password_change/ [name='password_change']
  # accounts/ password_change/done/ [name='password_change_done']
  # accounts/ password_reset/ [name='password_reset']
  # accounts/ password_reset/done/ [name='password_reset_done']
  # accounts/ reset/<uidb64>/<token>/ [name='password_reset_confirm']
  # accounts/ reset/done/ [name='password_reset_complete']
  ```

- Redirect: `settings.py`

  ```python
  # Redirect to home URL after login (Default redirects to /accounts/profile/)
  LOGIN_REDIRECT_URL = '/'
  ```

- Login required

  ```python
  from django.contrib.auth.decorators import login_required

  # If the user is not logged in, redirect to the login URL defined in the settings.LOGIN_URL
  @login_required
  def my_view(request):
      ...
  ```

  ```python
  from django.contrib.auth.mixins import LoginRequiredMixin

  class MyView(LoginRequiredMixin, View):
    login_url = '/login/' # optional
    redirect_field_name = 'redirect_to' # optional
  ```

- Permissions

  - Models

    ```python
    class BookInstance(models.Model):
        ...
        class Meta:
            ...
            # tuple containing the name and display value
            permissions = (("can_mark_returned", "Set book as returned"),)
    ```

  - Templates

    ```html
    {% if perms.catalog.can_mark_returned %}
    <!-- We can mark a BookInstance as returned. -->
    <!-- Perhaps add code to link to a "book return" view here. -->
    {% endif %} {% if user.is_staff %}
    <!-- content is for staff only -->
    {% endif %}
    ```

  - Views

    ```python
    from django.contrib.auth.decorators import permission_required

    # logged-in user with a permission violation: redirects to login(Status 302)
    @permission_required('catalog.can_mark_returned')
    @permission_required('catalog.can_edit')
    def my_view(request):
        ...
    ```

    ```python
    from django.contrib.auth.decorators import login_required, permission_required

    @login_required
    @permission_required('catalog.can_mark_returned', raise_exception=True)
    def my_view(request):
    ...
    ```

    ```python
    from django.contrib.auth.mixins import PermissionRequiredMixin

    class MyView(PermissionRequiredMixin, View):
        permission_required = 'catalog.can_mark_returned'
        # Or multiple permissions
        permission_required = ('catalog.can_mark_returned', 'catalog.can_edit')
        # Note that 'catalog.can_edit' is just an example.The catalog application doesn't have such permission!
    ```

## Django Tutorial Part9: Working with forms

lookups files and docs.

## Django Tutorial Part10: Testing a Django web application

```python
class YourTestClass(TestCase):
    def setUp(self):
        # Setup run before every test method.
        pass

    def tearDown(self):
        # Clean up run after every test method.
        pass

    def test_something_that_will_pass(self):
        self.assertFalse(False)

    def test_something_that_will_fail(self):
        self.assertTrue(False)
```

```html
<!-- Django will discover  tests under the current working directory in any file named with the pattern test*.py -->
catalog/ /tests/ __init__.py test_models.py test_forms.py test_views.py
```

```shell
python3 manage.py collectstatic
python3 manage.py test
python3 manage.py test catalog.tests.test_models.YourTestClass.test_one_plus_one_equals_two
```

## Django Turorial Part11: Deploying Django to production

`settings.py`

```python
DEBUG = False

# Read SECRET_KEY from an environment variable
import os
SECRET_KEY = os.environ['SECRET_KEY']

# OR

# Read secret key from a file
with open('/etc/secret_key.txt') as f:
    SECRET_KEY = f.read().strip()
```

- ALLOWED_HOSTS
- DATABASES
- STATIC_ROOT and STATIC_URL
- ...

## Django web application security

- Cross site scription (XSS)
- Cross site request forgery (CSRF)
- SQL injection
- Clickjacking
- SSL/HTTPS

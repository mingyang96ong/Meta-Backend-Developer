# Django (Backend Web Service Framework)

## Short Recap
1. Model View Template (MVT) Architecture
   1. Variation from Model View Controller (MVC) Architecture
   2. Different from MVC, View in MVT is the controller and Template in MVT is the View.
   3. URL Dispatcher component works like the controller in MVC
2. URL Routing
   1. Mainly stored in `urls.py` in the project-level
   2. Common convention would create a `urls.py` to be stored in app-level and imported into project-level `urls.py`
   3. Mainly uses `django.urls.path`, `django.urls.include` and `django.urls.re_path`
   4. In the `urlpatterns` variable, add the `path('resource', some_view)` or `path('somepath', include('myapp.urls'))`
3. Views
   1. Manage the logic and display of the template
   2. There are two types of views: Function-based or class based
   3. Both requires a mandatory argument of `request` of type `django.http.HttpRequest`
   4. Class Based views defines view in the respective http method (get, post, put, delete) found in `django.views.View`
4. Parameter passing
   1. Can be done in two ways: URL Path parameter, Query Parameters and Body Parameters
      1. URL Path Parameter
         1. Requires to change in `urls.py` in app-level
         2. Assuming we want to take in id, `path('about/<int:id>', views.myview)`
         3. We need define our view `myview` as `def myview(request, id):`
         4. Default converter will be `str`
         5. Other types:
            1. `int`: Matches zero or any positive integer
            2. `slug`: Matches any slug string consisting of ASCII letters or numbers, including hyphen and underscore
            3. `uuid`: Matches a formatted UUID. For example: 075194d3-6885-417e-a8a8-6c931e272f00 and returns a UUID instance.
            4. `path`: Matchs any non-empty string and includes the path separator `'/'` 
      2. Query Parameter
         1. Stored in the `django.http.HttpRequest` `GET` attribute
         2. i.e `id = request.GET['id']`
      3. Body Parameter
         1. Stored in the `django.http.HttpRequest` `POST` attribute
         2. i.e `id = request.POST['id']`
5. Error Handling
   1. Set up `handler400`, `handle403`, `handler404`, `handler500` variable in `urls.py` in the project level
   2. You use `django.http.HttpResponseNotFound` to return a 404 response
   3. Debug = True will not be able to see the error message set for error handling
   4. ALLOWED_HOSTS cannot be empty
   5. i.e `handler404 = 'myprojects.views.handler404'`
6. Models (Object Relatioship Mapping)
   1. Each model class creates a new table.
   2. Each table adds an `id` column by default
   3. The model class has to inherit from `django.dbs.models.Model`
   4. Attributes of model class represents a table column.
   5. Example
    ```python
    from django.db import models
    class User(models.Model):
        first_name = models.CharField(max_length=30)
        last_name = models.CharField(max_length=30)
        def __str__(self): # This will change the way it display in django admin site
            return first_name + ' ' + last_name
    ```
   6. CRUD Operation
      1. Creating a `User` object/record
        ```python
        new_user = User(id=1, first_name='John', last_name='Jones')
        new_user.save()
        ```
      2. Read a `User` object
        ```python
        new_user = User.objects.get(id=1)
        ```
      3. Update a `User` object/record
        ```python
        new_user = User.objects.get(id=1)
        new_user.last_name = "Smith"
        new_user.save()
        ```
      4. Delete a `User`object/record
        ```python
        new_user = User.objects.filter(id=1).delete()
        ```
   7. Additional Function in` models.objects`
      1. `models.objects.all()` -> Get all items
      2. `models.objects.select_related('foreign_key_field').all()` -> Get all items and retrieve related foreign key objects in one sql
      3. `models.objects.get(pk=key)` -> Get a single item with pk equals to `key`
      4. `models.objects.all().order_by('some_field')` -> Order items by some field ascendingly, descending (`-some_field`), it can take in multiple fields to order by multiple field
      5. `models.objects.filter(cond)` -> Get all list that meets the conditions
         1. This is a rather broad topic. A simple usage example
            ```python
            mymodel_objs = mymodel.objects.filter(field1='John')
            ```
         2. You can access inner foreign object with `__` double undercore
            ```python
            mymodel_objs = mymodel.objects.filter(foreign_obj__name='John') # Get items with foreign obj's name field = `John`
            ```
         3. You can even use special filter conditions
            1. `somefields__startswith` -> Fields starts with some str
            2. `somefields__istartswith` -> Fields starts with some str (case insensitive)
            3. `somefields__lte` -> Fields less than equal to
            4. `somefields__contains` -> Fields contains certain str
            5. `somefields__icontains` -> Fields contains certain str (case insensitive)
   8. Migrations
      1. Allow tracking of databases changes history
      2. Allow syncing of databases table and model objects
   9.  Provided `makemigrations`, `migrate`, `showmigrations`, `sqlmigrate` functions
      1. `makemigrations` creates the migration scripts to apply to the database
      2. `migrate` executes the unapplied migration scripts and it can also revert the change by passing a particular migration script
         1. i.e `python manage.py migrate <app_name> 0001_initial` -> Revert to `0001_initial.py`
         2. i.e `python manage.py migrate <app_name> zero` -> Revert to initial status without any tables
      3. `showmigrations` shows the past history of applied and unapplied migrations
      4. `sqlmigrate` show sql code of a particular migration script
         1. i.e `python manage.py sqlmigrate demoapp 0001_initial`
7. Forms (Built-in django)
   1. `django.forms` provide multiple different forms for usage
   2. `django.forms.ModelForm` allows form to work hand in hand with  `django.models.Model` 
      1. Define `django.models.Model`
      2. Followed by defined another inner class `Meta` in `django.forms.ModelForm`
         1. Indicate its `model` attributes and `fields` attributes
      3. In the view, simply pass in the `django.http.HttpRequest` `POST` into the defined `django.forms.ModelForm` and call the save function to save into the database.
8. Django Admin
   1. Create an account with `python manage.py createsuperuser`
   2. It should already be included in the `INSTALLED_APPS` of `setting.py`
   3. We can edit permission of users that logged into the Django Admin
      1. We will first customise the User permission by unregister the `django.contrib.auth.models.User` from `django.contrib.admin`
      2. We can create the new User class by inheriting from `django.contrib.auth.admin.UserAdmin` and add a `@admin.register(User)` decorator
      3. From here on, we can choose to disable field for all users or only some groups of user
         1. To disable a field from all users, set the class variable `readonly_fields` to prevent certain fields in the new User class to be editted
         2. To disable a field from some user, we need to change the instance function `get_form(self, request, obj=None, **kwargs)` by checking the user's permission in this function and disable the fields accordingly. i.e `super().get_form(request, obj, **kwargs).base_fields['username'].disabled = True` will disable the user from editting `username`
   4. We can also edit the data displayed by the following two methods
      1. We can edit the `__str__` in the model class
      2. We can also set the `list_display` class variable for the `django.contrib.admin.ModelAdmin` for the particular model class with the `@admin.register(Person)` decorator
   5. We can also change the search function of a model class by changing the `search_fields` class variable of `django.contrib.admin.ModelAdmin` of the model class
   6. Permission of each model class are named as `<app>.<change model>`. i.e `myapp.add_mymodel`, `myapp.change_mymodel`, `myapp.delete_mymodel`, `myapp.view_mymodel`
      1. These permssions can be used to check the user's permisssion with `has_perm` function. i.e `request.user.has_perm('my_app.view_mymodel')`
      2. However, this means `request.user` has to be authenicated/login. These are some useful functions
         1. `@login_required` - Decorator that requires user to be logged in (imported from `django.contrib.auth.decorators`)
         2. `request.user.is_anonymous` - Function to check if the user is logged in
         3. `@user_passes_test(func(request.user) -> bool, Optional login_url)` - Decorator to enforce view to check for custom permission otherwise redirect to login url (imported from `django.contrib.auth.decorators`)
         4. `@permission_required('myapp.change_mymodel')` - Decorator that requires user to have the permission for the view function to trigger (imported from `django.contrib.auth.decorators`)
         5.  `PermissionRequiredMixin` class and `permission_required` class attribute - Decorator to enforce user to have permission for **class** based view (imported from `from django.contrib.auth.mixins` )
         6.  `{% if user.is_authenticated % } {render html} {% endif %}` - Template way of checking logged-in users and render the template
         7.  `{% if perms.change_mymodel % } {render html} {% endif %}` -  Template way of checking users with permission `change_mymodel` and render the template
         8.  `url(r'^users_only/', login_required(myview))` - URL way of enforcing logged-in users (imported from `url` from `django.conf.urls` and `login_required` from `django.contrib.auth.decorators`)
         9.  `url(r'^category/', permission_required('myapp.change_mymodel'), login_url='login')(myview)` - URL way of enforcing permission (imported `url` from `django.conf.urls` and `permission_required` from `django.contrib.auth.decorators`)
9.  Database Options Setup
   1.  The setup is stored in the `DATABASES` variable in `settings.py`
   2.  Can view the mysql recap below
10. Templates
    1.  Allows dynamic content of html. 
    2.  You can declare where to find the templates in `settings.py` of `TEMPLATES[0]['DIRS']`
    3.  `{{ var }}` - variable tag for `var`, where we can provide context variable to `render` in view function
    4.  `{% if %}` `{% else %}` `{% endif %}` - conditional tag
    5.  `{% for %}` `{% endfor %}` - for loop to loop template over list of elements
    6.  `{{ var | filter name }}` - Process variable with filter. Example filters as below
        1.  `{{ value|default:"nothing" }} ` - Set default value to `nothing`
        2.  `{{ words|join:" _ " }}` - Similar to python `str.join('_')`
        3.  `{{ name|length }}` - Return length of sequence
        4.  `{{ list|first }}` - Return first item of the list
        5.  `{{ list|last }}`- Return last item of the list
        6.  `{{ name|lower }}` - Convert string to lower
        7.  `{{ name|upper }}` - Convert string to upper
        8.  `{{ string|title }}`  - Making word to start with uppercase and remaining characters as lowercase
        9.  `{{ string|wordcount }} ` - Return the number of words
        10. `{{ string|add:var }}` - Allows string and `var` string variable to be concatenated together
    7.  `{% load static %}` and `{% static 'path_to_file' %}` - To enable static contents, also have to declare `STATICFILES_DIRS` in `settings.py` for the list of static files directory
    8.  `{% comment "optional_note" %}` `{% endcomment %}` -  Comment block in django html template
    9.  `{% csrf_token %}` - tag is used in a form template as protection to prevent Cross Site Request Forgeries (CSRF)
    10. `{% block content %}` - Defines a block that can be overridden by child templates
    11. `{% include 'somehtml.html' %}` - Loads a template and renders it
    12. `{% extends 'somehtml.html' %}` `{% block %}` `{% endblock %}` - Inherit another template and renders the current template within the `block` tags
    13. `{% with var = 5 %}` `{% endwith %}`- set local variable
    14. `{% url 'home' pk=item.pk %}` - Auto link the url with path name variable, you can pass path parameters into url
11. Testing Django
    1.  Inherit from `django.test.TestCase`
    2.  Pretty much very similar to pytest, define a function and use `assert` to test
    3.  We can also setup test data by defining `setUp(self)` 
    4.  In command line, run `python3 manage.py test` to run all tests
    5.  We can run specific testcase package `python3 manage.py test testcase_package`
    6.  We can run specific testcase `python3 manage.py test testcase_package Testcase.method`
    7.  For API testing, you need to create the database from scratch. Meaning even user account has to recreate.
    8.  Supposed we want to group tests together in one folder, we need to add an `__init__.py`
12. Pagination -> Reduce the number of items returned (Improved bandwidth, Lesser load on Database)
    1. `django.core.paginator.Paginator` -> Takes in list of objects and per_page argument. Call `paginator.page(number=page_number)` to return the expected items based on the page number. Basically, handles the indexing of the items per page and return accordingly to the page number in the SQL. Hence, lesser load on the database
    2. `django.core.paginator.EmptyPage` -> An exception when there is insufficient items in list to return. Therefore it will be empty page.
    3. To setup pagination in `django`, you need add default `pagination_class` to a view
    4. If you want to add a default pagination class to your whole project, you can add it to `settings.py`
13. Static Files
    1.  By default, this `django.contrib.staticfiles` is included in `INSTALLED_APPS` of `settings.py`
    2.  `STATIC_URL` - The url where static contents will be served
    3.  `STATIC_ROOT` - The directory to copy your static files to after using `python manage.py collectstatic`
    4.  `STATICFILES_DIRS` - The directories to find your static files
    5.  By default, if `STATICFILES_DIRS` is not declared, the template will look for static files in the `<app_folder>/static/<path in your template>` location

## Short mysql recap
1. Default port: 3306
2. For python 3 onwards, you need to install `pymysql` on top of `mysqlclient`. To use it in django project, you need to import it in `__init__.py` in the project folder. Additionally, run `pymysql.install_as_MySQLdb()` for django use the library client.
3. Create a mysql user account command: `CREATE USER 'admindjango'@'localhost' IDENTIFIED BY 'employee@123!' ;`
4. Granting all the permission to an account mysql command: `GRANT ALL ON *.* TO 'admindjango'@'localhost';`
5. Flush permission mysql command: `FLUSH PRIVILEGES;`
6. Change the `DATABASES` in `settings.py`.
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'menu_items', # Database name
        'HOST': 'localhost',
        'USER': 'admindjango',
        'PASSWORD': 'employee@123!',
        'PORT': '3306',
   }
}
```
7. You need to also include the `mysqlclient` version in `pymysql`

# Setup django
```bash
# venv way of creating env
python -m pip install virtualenv
python -m venv djangoenv

source djangoenv/bin/activate
# deactivate -> to get out of virtual env

python -m pip install django


# pipenv way of creating env
python -m pip install pipenv # If pipenv not installed.
pipenv install django
pipenv shell # To start the virtual env



# Start Project as 'myproject'
django-admin startproject 'myproject' .

# Start app as 'myapp'
# Run command at the parent directory of project
python manage.py startapp 'myapp'
```


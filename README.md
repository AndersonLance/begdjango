# Beginner's Guide to Django

The Complete Beginner's Guide to Django:
* [Part 1](https://simpleisbetterthancomplex.com/series/2017/09/04/a-complete-beginners-guide-to-django-part-1.html)
* [Part 2](https://simpleisbetterthancomplex.com/series/2017/09/11/a-complete-beginners-guide-to-django-part-2.html)

## Notes - Part 1

### Benefits of Django
* Based on Python, so you can benefit from a wide range of open source libraries out there. The (Python Package Index)[https://pypi.org/] repository hosts over 116k packages.

### Basic Setup

1. Install Python
	* Use Homebrew
2. Install Virtualenv
	* Use pip
3. Install Django inside a virtual environment
	* cd into directory for a project, then: `virtualenv venv -p python3`
4. Activate virtual environment
	* `source venv/bin/activate`
5. Run Python Development Server
    * `python manage.py runserver`

#### So what did we just do?

We created a special folder called venv, and it contains a copy of Python inside this folder. After we activated the venv environment, when we run the `python` command, it will use our local copy, stored inside venv, instead of the other one we installed earlier.

Another thing of note, is that the pip program is installed as well, and when we use it to install a Paython package, like Django, it will be installed inside the venv environment.

Note that when we have the venv activated, we will use the command `python` (instead of python3) to refer to Python, and just `pip` (instead of pip3) to install packages.

To deactivate (exit/quit) a venv, use `deactivate`.

#### Install Django

Inside of the running venv, type `pip install django`

### Starting a New Project

To start a new Django project, type `django-admin startproject myproject`
(Substitue 'myproject' with whatever project name you would like to use.)

The command-line utility django-admin is automatically installed with Django. After running this command, it will automatically create the basic folder structure for your project.

#### Initial Project Structure

Composed of 5 files:

* manage.py: A shortcut to use the django-admin CL utility. It's used to run management commands related to our project. We will use it to run the development server, run tests, create migrations and much more.
* __init__.py: This empty file tells Python that this folder is a Paython package.
* settings.py: Contains all the project's configuration.
* urls.py: Responsible for mapping the routes and paths in the project. For example, if you want to show something in the URL /about/, you have to map it here first.
* wsgi.py: Simple gateway interface used for deployment.

#### Run Development Server

Django comes with a web server. To run it, make sure you have activated your virtual environment, and that you are in the actual project directory containing the manage.py file. Then type: `python manage.py runserver`

### Django Apps

Create an app. Inside of running venv, type: `django-admin startapp boards`

This will create a directory called `boards` inside of your project directory. In it, you will find the following:

* migrations/: Folder where Django stores some files to keep track of the changes you make in the models.py file, to keep the database and models.py synchronized.
* admin.py: Config file for a built-in Django app called Django Admin.
* apps.py: Config file of the app itself.
* models.py: Here is where you define the entities of your web app. The models are translated automatically by Django into database tables.
* tests.py: This file is used to write unit tests for the app.
* views.py: This file is where we handle request/response cycle of the web app.

After creating app, let's configure the project so we can use it.

#### Configure Project for Use

Open `settings.py` and try to find the `INSTALLED_APPS` variable. You will see that Django already comes with several built-in apps installed. These handle common functionalities needed for web apps. For now, let's add the new app we just created. Append a new item inside of the `INSTALLED_APPS` variable, called 'boards' (since boards is the name of the app we just created using the djang-admin startapp command)

    INSTALLED_APPS = [
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',

        'boards',
    ]

### Hello World!

Let's write our first view.

Open the `views.py` file, and add the following code:

    from django.http import HttpResponse

    def home(request):
        return HttpResponse('Hello, World!)

Views are Python functions that receive an `HttpRequest` object and return an `HttpResponse` object. Receive a request as a parameter and returns a response as a result. That's the flow to keep in mind.

So here, we defined a simple view called `home` which simply returns a message saying `Hello, World!`

Now we have to tell Django when to serve this view. It's done inside the `urls.py` file:

    from django.contrib import admin
    from django.conf.urls import url

    from boards import views

    urlpatterns = [
        url(r'^$', views.home, name='home'),
        url('admin/', admin.site.urls),
    ]

*IMPORTANT: By default, Django seems to use `path` instead of `url`. In the above code I changed the so it uses url instead of path. The syntax is slightly different, and I'm not sure why. Don't for get to change the 'from' line as well, otherwise you are importing path and using url, or vice-versa.*

Django works with regex to match the requested URL. For home view, using the `^$` regex, which will match an empty path, which is the homepage (this url: http://127.0.0.1:8000). Now, if you browse to that URL you will see a Hello World message.

## Notes - Part 2

### Create Models

Create models inside the `boards/models.py` file. For this project, we will create a model for a Board, a Topic, and a Post. We will use Django's built-in authentication (auth) function and import its `user` model. When creating a model, you give it a name, and tell it what fields it will use:

    from django.db import models
    from django.contrib.auth.models import User

    # Create your models here.
    class Board(models.Model):
        name = models.CharField(max_length=30, unique=True)
        description = models.CharField(max_length=100)


    class Topic(models.Model):
        subject = models.CharField(max_length=255)
        last_updated = models.DateTimeField(auto_now_add=True)
        board = models.ForeignKey(Board, related_name='topics', on_delete=models.DO_NOTHING)
        starter = models.ForeignKey(User, related_name='topics', on_delete=models.DO_NOTHING)


    class Post(models.Model):
        message = models.TextField(max_length=4000)
        topic = models.ForeignKey(Topic, related_name='posts', on_delete=models.DO_NOTHING)
        created_at = models.DateTimeField(auto_now_add=True)
        updated_at = models.DateTimeField(null=True)
        created_by = models.ForeignKey(User, related_name='posts', on_delete=models.DO_NOTHING)
        updated_by = models.ForeignKey(User, null=True, related_name='+', on_delete=models.DO_NOTHING)

*IMPORTANT: The original tutorial did not have the `on_delete=models.DO_NOTHING` property for each of the ForeignKey fields. However, when trying to migrate the models in the next step I was receiving errors. After some research, it looks like the on_delete property is required for all ForeignKey fields. This was not specified in the tutorial at all, so maybe he's using an older version? I referenced this Stack Overflow [post](https://stackoverflow.com/questions/44026548/getting-typeerror-init-missing-1-required-positional-argument-on-delete). There are more options than DO_NOTHING, here are the [Django Docs](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey) about it.*

All models are subclasses of the `django.db.models.Model` class. Each class is transformed into a database table. Each field is represented by instances of `django.db.models.Field`

The fields `CharField`, `DateTimeField`, etc. are all subclasses of `django.db.models.Field` and they come with the Django core. In this example, we are using `CharField, TextField, DateTimeField, and ForeignKey` fields, but Django offers a wide range of options such as `IntegerField, BooleanField, DecimalField` and others.

### Migrate Models

The next step is to create the database so we can start using it.

Activate virtual environment, and navigate to the folder where `manage.py` is, and type:

    python manage.py makemigrations

This will create a file named `0001_initial.py` inside the `boards/migrations` directory. It represents the current state of our app's models.

The migration files are translated into SQL statements. If you are familiar with SQL, you can run the following command to inspect the SQL instructions that will be executed in the database:

    python manage.py sqlmigrate boards 0001

### Apply Migration to Database

    python manage.py migrate

Database is now ready for use.

### Experimenting with the Models API

One advantage of Python is the interactive shell. You can quickly try things out and experiment with libraries and APIs.

You can start a Python shell within our project using the manage.py utility:

    python manage.py shell

This is similar to calling the interactive console just by typing `python`, except when we use `python manage.py shell`, we are adding our project to the `sys.path` and loading Django. That means we can import our models and any other resource within the project and play with it.

Start by importing the `Board` class:

    from boards.models import Board

To create a new board object, do the following:

    board = Board(name='Django', description='This is a board about Django.')

To persist the object in the database, call the `save` method:

    board.save()

The `save` method is used to both *create* and *update* objects. Here Django created a new object because the `Board` instance had no `id`. After saving it for the first time, Django will set the id automatically:

    board.id

You can access the rest of the fields as Python attributes:

    board.name
    board.description

To update a value:

    board.description = 'Django discussion board.'
    board.save()

Every Django model comes with a special attribute called a `Model Manager`. It is used mainly to execute queries in the database. For example, we could use it to directly create a new `Board` object:

    board = Board.objects.create(name='Python', description='General discussion about Python.')

    board.id
    board.name
    board.description

You can use `objects` to list all existing boards in the database:

    Board.objects.all()

To exit the interactive console, type:

    exit()

You can use the `Model Manager` to query the database and return a single object:

    django_board = Board.objects.get(id=1)

    django_board.name

You can use the `get` method with any model field, but preferrably use a field that can uniquely identify an object. Otherwise, you may return more than one object, which will cause an error.

**NOTE: The queries are case sensitive.**

### Summary of Model's Operations

In these examples, uppercase `Board` refers to the class, while lowercase `board` refers to an instance (or object) of the `Board` model class:

| Operation | Code Sample |
| --- | --- |
| Create an object without saving | `board = Board()` |
| Save an object (create or update) | `board.save()` |
| Create and save an objeect in the database | `Board.objects.create(name='...', description='...')` |
| List all objects | `Board.objects.all()` |
| Get a single object, id'd by field | `Board.objects.get(id=1)` |

### Views, Templates, and Static Files

Right now, there is a simple view called home being displayed on our server. (urls.py and boards/views.py) We can use this as a starting point. What we want to do is to display a list of boards in a table alongside some other information.

The first thing to do is to import the `Board` model and list all the existing boards:

boards/views.py:

    from django.http import HttpResponse
    from .models import Board

    def home(request):
        boards = Board.objects.all()
        boards_names = list()

        for board in boards:
            boards_names.append(board.name)
        
        response_html = '<br>'.join(boards_names)

        return HttpResponse(response_html)

And the result would be a simple HTML page that just brings back the names of the two boards we created earlier: Django and Python. But this is not how we really want to be rendering HTML, we should use the `Django Template Engine`.

Create a new folder named `templates` alongside with the `boards` folder. Within the new templates folder you just created, create a file named `home.html`:

    <!DOCTYPE html>
    <html>
        <head>
            <meta charset="utf-8">
            <title>Boards</title>
        </head>
        <body>
            <h1>Boards</h1>

            {% for board in boards %}
            {{ board.name }} <br>
            {% endfor %}

        </body>
    </html>

In this file, we mix raw HTML with some special tags `{% for ... in ... %}` and `{{variable}}`. They are part of the Django Template language. This shows how to iterate over a list of objects using a `for`. The `{{ board.name }}` renders the name of the board in the HTML template, generating a dynamic HTML document.

Before using this HTML page, we have to tell Django where to find our application's templates.

Open `settings.py` and search for the `TEMPLATES` variable and set the `DIRS` key to `os.path.join(BASE_DIR, 'templates'):

    TEMPLATES = [
        {
            'BACKEND': 'django.template.backends.django.DjangoTemplates',
            'DIRS': [
                os.path.join(BASE_DIR, 'templates')
            ],
            'APP_DIRS': True,
            'OPTIONS': {
                'context_processors': [
                    'django.template.context_processors.debug',
                    'django.template.context_processors.request',
                    'django.contrib.auth.context_processors.auth',
                    'django.contrib.messages.context_processors.messages',
                ],
            },
        },
    ]

This is finding the full path of your project directory and appending '/templates' to it.

Now we can update the `home` view
boards/views.py:

    from django.shortcuts import render
    from .models import Board

    def home(request):
        boards = Board.objects.all()
        return render(request, 'home.html', {'boards' : boards})
        boards_names = list()

Now, you can improve the HTML template to use a table instead in the `templates/home.html` file.

    <!DOCTYPE html>
    <html>
    <head>
        <meta charset="utf-8">
        <title>Boards</title>
    </head>
    <body>
        <h1>Boards</h1>

        <table border="1">
            <thead>
                <tr>
                    <th>Board</th>
                    <th>Posts</th>
                    <th>Topics</th>
                    <th>Last Post</th>
                </tr>
            </thead>
            <tbody>
                {% for board in boards %}
                    <tr>
                        <td>
                            {{ board.name }}<br>
                            <small style="color: #888">{{ board.description }}</small>
                        </td>
                        <td>0</td>
                        <td>0</td>
                        <td></td>
                    </tr>
                {% endfor %}
            </tbody>
        </table>
    </body>
    </html>

#### Testing the Homepage

Let's write our first test. For now we will work with the `tests.py` file inside the `boards` app:

boards/tests.py:

    from django.urls import reverse
    from django.test import TestCase

    # Create your tests here.
    class HomeTests(TestCase):
        def test_home_view_status_code(self):
            url = reverse('home')
            response = self.client.get(url)
            self.assertEquals(response.status_code, 200)

This is a very simple test that tests the status code of the response. Status code 200 means success. You can check the status code of the response in the console:

`[13/Aug/2020 19:33:22] "GET / HTTP/1.1" 200 1065`

If there were an error, exception, syntax error, etc. Django would reaturn a status code 500 instead, which means internal server error. Now imagine an app with 100+ views. If we wrote this test for all of our views, with just one command you would be able to test if all of the views were returning a success code. Without automated tests, you'd have to check each page one by one.

To execute Django's test suite:

    python manage.py test

Now we can test to see if the correct view function was returned for the requested URL.

boards/tests.py:

    from django.urls import reverse
    from django.urls import resolve
    from django.test import TestCase
    from .views import home

    # Create your tests here.
    class HomeTests(TestCase):
        def test_home_view_status_code(self):
            url = reverse('home')
            response = self.client.get(url)
            self.assertEquals(response.status_code, 200)
        def test_home_url_resolves_home_view(self):
            view = resolve('/')
            self.assertEquals(view.func, home)

This test makes use of the `resolve` function. Django uses it to match a requested URL with a list of URLs listed in the `urls.py` module. This will make sure the URL `/` is returning the home view. Test it again:

    python manage.py test

#### Static Files Setup

Static files are the CSS, JS, Fonts, Images, etc. As-is, Django doesn't serve those files, except during the development process. However, Django does provide some features to help us manage the static files: Those features are available in `django.contrib.staticfiles` app already listed in the `INSTALLED_APPS` configuration.

We can easily add Bootstrap 4 to the project. In the project root dir, alongside boards and templates, create a new folder named `static` and within that folder create another folder called `css`. Download Bootstrap 4 (Compiled CSS and JS). Extract bootstrap.min.css file into the newly created css folder in our project. Next, instruct Django where to find the static files.

Open `settings.py`, scroll to the bottom and just after the `STATIC_URL`, add:

    STATICFILES_DIRS = [
        os.path.join(BASE_DIR, 'static'),
    ]

Same thing as the `TEMPLATES` dir earlier.

Now, load the static files into our template

templates/home.html:

    {% load static %}<!DOCTYPE html>
    <html>
    <head>
        <meta charset="utf-8">
        <title>Boards</title>
        <link rel="stylesheet" href="{% static 'css/bootstrap.min.css' %}">
    </head>
    <body>
        <!-- body suppressed for brevity ... -->
    </body>
    </html>

First, we are loading the static files template tags by using `{% load static}` Then, the template tag `{% static %}` is used to compose a URL for where the resource lives.

The `{% static %}` tag uses the `STATIC_URL` configuration set in the `settings.py` file.

Now, if you run the server and refresh the page, you will see that the Bootstrap CSS is being applied. Now you can manage the home template to take advantage of the Bootstrap CSS classes.

templates/home.html:

    {% load static %}<!DOCTYPE html>
    <html>
    <head>
        <meta charset="utf-8">
        <title>Boards</title>
        <link rel="stylesheet" href="{% static 'css/bootstrap.min.css' %}">
    </head>
    <body>
        <div class="container">
        <ol class="breadcrumb my-4">
            <li class="breadcrumb-item active">Boards</li>
        </ol>
        <table class="table">
            <thead class="thead-inverse">
            <tr>
                <th>Board</th>
                <th>Posts</th>
                <th>Topics</th>
                <th>Last Post</th>
            </tr>
            </thead>
            <tbody>
            {% for board in boards %}
                <tr>
                <td>
                    {{ board.name }}
                    <small class="text-muted d-block">{{ board.description }}</small>
                </td>
                <td class="align-middle">0</td>
                <td class="align-middle">0</td>
                <td></td>
                </tr>
            {% endfor %}
            </tbody>
        </table>
        </div>
    </body>
    </html>

Up to this point, we are adding new boards using the interactive console (`python manage.py shell`), but there is a better way: The Django Admin Interface.

### Django Admin



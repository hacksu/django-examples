# Introduction to Django

## Installing Django

### Mac/Unix
1. Download [Django-1.4.2.tar.gz](https://www.djangoproject.com/download/1.4.2/tarball/)
2. Run the following commands in terminal:

```
tar xzvf Django-1.4.2.tar.gz
cd Django-1.4.2
sudo python setup.py install
```

### PC
1. Download and install Python 2.7 if you dont have it. (http://www.python.org/ftp/python/2.7.3/python-2.7.3.msi)
2. Download Django (https://github.com/django/django/zipball/master)
3. Copy django folder into C:\Python27\Lib\site-packages\
4. Right click on “My Computer” and select Properties.
5. Go to Advanced, then Environment Variables at the bottom.
6. Edit “Path” and at the end add the following:

```
;C:\Python27;C\Python27\scripts;C:\Python27\Lib\site-packages\django\bin
```


##### Test to find out if it was a great success!
In cmd (you can find this program from the start menu), type

```
python
```

and press enter. The python shell should appear. Enter

```
import django
```

and then

```
django.get_version()
```

This should return

```
1.6
```

YAY

Note: exit() will get you out of the shell

## Once you have Django installed

### Creating your first project

in terminal enter:

    django-admin.py startproject mysite

This will create a my site directory in your current directory. In the my site directory, the following has been created:

    mysite/
        manage.py
        mysite/
            __init__.py
            settings.py
            urls.py
            wsgi.py

To run your Django server, enter the following in your my site folder:

    python manage.py runserver 

Note, by default it is hosted at http://127.0.0.1:8000/

Take a look at the "Welcome to Django" page :)

To change the port, you can pass it when you call the run run server command:

    python manage.py runserver 8080

So that is pretty sweet, you just ran your first Django server! :)

### Setting up the database

Open up mysite/settings.py. Find the DATABASE object, and we will use sqlite3 in this example.

Lets make the following changes to settings.py (2 options):

```python
'ENGINE' : 'django.db.backends.sqlite3',
'NAME' : '/Users/username/some/path/database.db', 
```

OR

```python
import os

SITE_ROOT = os.path.dirname(os.path.realpath(__file__))
...
'ENGINE' : 'django.db.backends.sqlite3',
'NAME' : os.path.join(SITE_ROOT, 'database.db'),
```

Now that we have the database setting complete, lets sync the database:

    python manage.py syncdb

### Creating the Models

Now, lets create an app. A project can have many apps.

    python manage.py startapp hacksu

The following directory will be created:

    hacksu/
        __init__.py
        models.py
        tests.py
        views.py

Lets give our app some models, open up the models.py file and add the following:

```python
from django.db import models

class Member(models.Model): 
    name = models.CharField(max_length=200)
    email = models.CharField(max_length=200)
    join_date = models.DateTimeField()

class Project(models.Model):
    name = models.CharField(max_length=200) 
    url = models.CharField(max_length=200)
    description = models.CharField(max_length=200)
    members = models.ManyToManyField(Member)
```

Cool, so you have created two models, now to link them. In settings.py, let's add 'hacksu' to INSTALLED_APPS in settings.py

Awesome. So now lets tell Django to include our app. 

    python manage.py sql hacksu

Now you will see the two create tables commands. That is super awesome. You never need to create a database manually again =)

To create those new model tables in your database, run:

    python manage.py syncdb    

Lets play with the python shell:

    python manage.py shell

Lets create a member:

    >>> from hacksu.models import Member
    >>> Member.objects.all()
    []

    >>> from django.utils import timezone
    >>> m = Member(name="Dawe", email="some@one.com", join_date=timezone.now())
    >> m.save()
    >> Member.objects.all()
    [<Member: Member object>]
    
    # That is not cool!

Lets fix our models, so that they give us human readable info. In hacksu/models.py
```python
class Member(models.Model):
    # ...
    def __unicode__(self):
        return self.name

class Project(models.Model):
    # ...
    def __unicode__(self):
        return self.name
```


Lets hop back into the django shell, and type:

    >>> from hacksu.models import Member

    >>> Member.objects.all()

    [<Member: Dawe>]

### Admin Features

So now we have done too much backend stuff. Let us look at the admin features. There are three steps needed:

1. Open up settings.py, and in INSTALLED_APPS, and uncomment 'django.contrib.admin'  
2. Run 'pyton manage.py syncdb'
3. In mysite/urls.py, uncomment the following 3 lines:
    
```python
from django.contrib import admin
admin.autodiscover()
...
url(r'^admin/', include(admin.site.urls)),
```

Now, run the server, by entering the following in terminal:

    python manage.py runserver

Visit: http://127.0.0.1:8000/admin/ and enter the admin username and password.

That is awesome, right. But you will see, Auth and Sites. Would you like to see your data models there too?

Ok, so lets open /hacksu/admin.py and add the following:

```python
from hacksu.models import Member, Project
from django.contrib import admin

admin.site.register(Member)
admin.site.register(Project)
```

This tells Django to add your models to the admin interface. 

To customize the template look and feel, go [here](https://docs.djangoproject.com/en/1.4/intro/tutorial02/#customize-the-admin-look-and-feel)

### Designing the URL's

In mysite/urls.py add the following: 

```python
#...
urlpatterns += patterns('hacksu.views',
    url(r'^$', 'index'),
)
```

That tells the server, that when someone requests root, they will be forwarded to index of our hacksu views. We need to add the functionailty there, so lets open hacksu/views.py

```python
from django.shortcuts import render_to_response
from hacksu.models import Member, Project

def index(request):
    members = Member.objects.all()
    projects = Project.objects.all()
    return render_to_response('hacksu/index.html', {'members': members, 'projects': projects})
```

We also need to create the html file, and link it up. So lets open up settings.py and set the TEMPLATE_DIRS
    
```python    
TEMPLATE_DIRS = (
    os.path.join(SITE_ROOT, 'templates'),
)
```

Lets create a templates folder in mysite, and create an index.html file inside there. 

Then inside that html file we can access our model objects and can embed python code :)

```html
<!DOCTYPE html>
    <head>
        <title>hacKSU Members</title>
    </head>
    <body>
        {% for member in members %}
            <p>{{ member.name }}</p>
            <p>{{ member.email }}</p>
            <p>{{ member.join_date }}</p>
        {% endfor %}

        {% if not members %}
            No members!
        {% endif %}
    </body>
</html>
```

### Get your app on Heroku!

Download and install Heroku, then launch your app from terminal:

1. heroku create
2. git push heroku master
3. heroku open


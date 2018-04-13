# Game-Server
Do this in the repo, should be done by ONE person (I’ll call this Person-A).
Go to the area you want your app, create the directory, then cd to it
	mkdir game_server
	cd game_server

IF you have this, I suggest doing it, if not just skip it.
Create a virtual environment to use (if you’re using one)
	virtualenv –p `which python3` .env

Activate your environment (this must be done every time you enter the directory)
	source env/bin/activate

EVERYONE must do this
Install Django
	python –m pip install django
	python –m pip install djangorestframework

Done by Person-A only
Create the app (using “gameapp” in this example
	Django-admin startproject gameapp

Done by Person-A 
Edit the .gitignore file for your project.  add to it:
    .env 

Do a git commit (you may have to do a git add if you just created the .gitignore file)
Add the gameapp directory, that should add everything under it.  Do another commit.
Push the changes up to github

Done by everyone
	Pull the repository from github (git pull).  Check that you see the new stuff

Done by Person-A, others will do it later on for other apps
Go into your Django project and create your first app
	python manage.py startapp user

Add the new app, and the Django_rest_famework  to the list of INSTALLED_APPS in the settings file within the gameapp
	…
	 'django.contrib.staticfiles',
  	'user',
	‘reset_framework’,

Due to the main app (gameapp) just being created and the new app (user) being added, there will be database migrations that must be performed
	python manage.py migrate

Add a route to your user API from the gameapp to your user API app by editing the gameapp/urls.py file, making it look similar to:
*gameapp/urls.py*
from user import views as uv
from django.urls import path

urlpatterns = [
    path('user/', uv.HomePageView().get),
    path('user/<str:uname>/', uv.HomePageView().get),
]

The user app must now have a GET method (and potentially others) that will return or process data when called.  Edit the user/views.py file to look like:
*user/views.py*
from django.http import HttpResponse
from rest_framework.views import APIView
import json

class HomePageView(APIView):
    def get(self, request=None, uname="test", format=None):
        data = { "user": uname, "key2": "value2" }
        return HttpResponse(json.dumps(data))

You should now have a fully functioning Django app that will return json data when called.  To run it:
	python manage.py runserver

You can test it by opening a web browser and going to:  http://localhost:8000/user/zelda

Stop the server, and work on the database.

Create the user table, which (for now) has an ID and the username.  Edit the User/models.py file to describe the database
*user/models.py*
from django.db import models

class User(models.Model):
    name = models.CharField(max_length=15)
    id = models.AutoField(primary_key=True)

Because the model has just changed, you will need to make a migration step that will change the database.  Django makes this easy
	python manage.py makemigrations

Then apply them as you did in the beginning
	python manage.py migrate

Now let’s change the ‘get’ method to actually look up the user

   # user/views.py
from django.http import HttpResponse
from rest_framework.views import APIView
from django.core.exceptions import ObjectDoesNotExist
import json
from user.models import User
from django.views.decorators.csrf import csrf_exempt

class HomePageView(APIView):
    def get(self, request=None, uname="test", format=None):
        try:
            found_user = User.objects.get(name=uname)
        except ObjectDoesNotExist as e:
            return HttpResponse(json.dumps({"status":"NoSuchUser"}), status=404)

        data = { "user": uname, "id": found_user.id }
        return HttpResponse(json.dumps(data))

This will try to get a user from the database,
Run the server again and try calling it again.  This time you should see that there is no such user.  That’s a little different than before.  So let’s take a look at getting data into the data base.  One way is from the command shell:

python manage.py shell
> from user.models import User
> u = User(‘zelda’)
> u.save()
> exit()

Now run your program, or if it is still running try reloading the webpage.  Hopefully it found Zelda this time.

Once this is working, Person-A should add any new files created to git, then commit them.

To make sure everything is in the repo, try:  “git status” and it will tell you anything you missed.



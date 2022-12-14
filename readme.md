python -m django --version
django-admin startproject namesite

startproject created:
namesite/ (root directory -- is a container for your project.)
manage.py (A command-line utility that lets you interact with this Django project in various ways.)
namesite/ (The inner namesite/ directory is the actual Python package for your project. Its name is the Python package name you’ll need to use to import anything inside it (e.g. namesite.urls).)
**init**.py (An empty file that tells Python that this directory should be considered a Python package.)
settings.py (Settings/configuration for this Django project.)
urls.py (The URL declarations for this Django project; a “table of contents” of your Django-powered site.)
asgi.py (An entry-point for ASGI-compatible web servers to serve your project.)
wsgi.py (An entry-point for WSGI-compatible web servers to serve your project)

python manage.py runserver
python manage.py runserver 8080 (to change the server’s port)
python manage.py runserver 0.0.0.0:8000 (to change the server’s IP)

Activating models:
python manage.py makemigrations
python manage.py migrate

Playing with the API:
python manage.py shell
from flights.models import \* !!!!!
Airport.objects.all()
Airport.objects.filter(city="New York")
Airport.objects.filter(city="New York").first()
Airport.objects.get(city="New York")

> > > python manage.py shell
> > > Python 3.9
> > > from flights.models import \*
> > > Airport.objects.all()
> > > <QuerySet [<Airport: New York (JFK)>, <Airport: London (LHR)>, <Airport: Paris (CDG)>, <Airport: Tokyo (NRT)>]>
> > > Airport.objects.filter(city="New York")
> > > <QuerySet [<Airport: New York (JFK)>]>
> > > Airport.objects.filter(city="New York").first()
> > > <Airport: New York (JFK)>
> > > Airport.objects.get(city="New York")
> > > <Airport: New York (JFK)>
> > > jfk = Airport.objects.get(city="New York")
> > > cdg = Airport.objects.get(city="Paris")
> > > cdg
> > > <Airport: Paris (CDG)>
> > > f = Flight(origin = jfk, destination = cdg, duration=435)
> > > f
> > > <Flight: None : New York (JFK) to Paris (CDG)>
> > > f.save()
> > > ^D -- exit from shell
> > > now exiting InteractiveConsole...

^C% -- stop server

python manage.py createsuperuser
Username (leave blank to use 'mary'): mary
Email address: uvk06@
Password: standart drroma
Password (again):
This password is too short. It must contain at least 8 characters.
This password is too common.
This password is entirely numeric.
Bypass password validation and create user anyway? [y/N]: y
Superuser created successfully.

next step: in admin.py in folder "flight" add admin

from django.contrib import admin
from .models import Flight, Airport
admin.site.register(Airport)
admin.site.register(Flight)

to urls.py add:
urlpatterns = [
path("", views.index, name="index"),
path("<int:flight_id>", views.flight, name="flight"),
]

to views.py add function:
def flight(request, flight_id):
flight= Flight.objects.get(pk=flight_id) !!!! pk -- primary key
return render(request, "flights/flight.html", {
"flight": flight
})

create flights.html in folder flights/templates/flights :
{% extends "flights/layout.html" %} {% block body %}

<h1>Your flight {{ flight.id }}</h1>
<ul>
  <li>Origin : {{ flight.origin }}</li>
  <li>Destination : {{ flight.destination }}</li>
  <li>Duration : {{ flight.duration }} min</li>
</ul>
{% endblock %}

in models.py add new class Passenger and his function:
class Passenger(models.Model):
first = models.CharField(max_length=64)
last = models.CharField(max_length=64)
flights= models.ManyToManyField(Flight, blank = True, related_name="passengers")
def **str**(self):
return f"{self.first} {self.last}"

now I have to do migrations(for create new model Passenger):
in terminal: stop server C,
python manage.py makemigrations
(Migrations for 'flights':
flights/migrations/0002_passenger.py - Create model Passenger)
python manage.py migrate (apply migrations to DB)  
(Operations to perform:
Apply all migrations: admin, auth, contenttypes, flights, sessions
Running migrations:
Applying flights.0002_passenger... OK)

add to admin.py about Passenger:
1.from .models import Flight, Airport, Passenger
2.admin.site.register(Passenger)

in views.py in func ---def flight add:
"passengers": flight.passengers.all()

in flight.html add:

<h2>Passengers</h2>
<ul>
  {% for passenger in passengers %}
  <li>{{passenger }}</li>
  {% empty %}
  <li>No passengers.</li>
  {% endfor %}
</ul>
in flight.html add link to all flight list:
<a href="{% url 'index' %}">Back to flight list</a>

change fragmrnt in index.html (add link):

<li>
    <a href="{% url 'flight' flight.id %}">
      Flight {{ flight.id }}: {{ flight.origin }} to {{ flight.destination }}
    </a>
  </li>
 
 I need new route for booking:
1) in urls.py in urlpatterns add:
 path("<int:flight_id>/book>", views.book, name="book"),
2) and in views.py new func:
def book(request, flight_id):
    if request.method =="POST":
        flight = Flight.objects.get(pk=flight_id)
        passenger = Passenger.objects.get(pk=int(request.POST['passenger']))
        passenger.flights.add(flight)
        return HttpResponseRedirect(reverse("flight", args=(flight.id,)))
3) and add import in views.py:
from django.http import HttpResponseRedirect
from django.urls import reverse
from .models import Flight, Passenger
4) create a form in flight.html:
<h2>Add Passenger</h2>
<form action="{% url 'book' flight.id %}" method="post">
  {% {% csrf_token %} %}
  <select name="passenger">
    {% for passenger in non_passengers %}
    <option value="{{ passenger.id }}">{{ passenger }}</option>
    {% endfor %}
  </select>
  <input type="submit" />
</form>
5)and in views.py change func flight -add:
"non_passengers": Passenger.objects.exclude(flights=flight).all()

add in flights/admin.py:

class FlightAdmin(admin.ModelAdmin):
list_display = ("id", "origin", "destination", "duration")
class PassengerAdmin(admin.ModelAdmin):
filter_horizontal=("flights",)
admin.site.register(Airport)
admin.site.register(Flight, FlightAdmin)
admin.site.register(Passenger, PassengerAdmin)

create new app "users":
python manage.py startapp users

in settings.py add 'users' ~
INSTALLED_APPS = [
'flights',
'users',
'django.contrib.admin',....

in urls.py add about users:
urlpatterns = [
path('admin/', admin.site.urls),
path('flights/', include('flights.urls')),
path('users/', include('users.urls')),
]

in folder users create new file urls.py

inside users/urls.py add:
from django.urls import path
from . import views
urlpatterns = [
path("", views.index, name="index"),
path("login", views.login_view, name="login"),
path("logout", views.logout_view, name="logout"),
]

now need to create this funcs: 1.index 2.login_view 3.logout_view in users/view.py:
from django.shortcuts import render
from django.http import HttpResponseRedirect
from django.urls import reverse
def index(request):
if not request.user.is_authenticated:
return HttpResponseRedirect(reverse('login'))
def login_view(request):
return render(request, "users/login.html")
def logout_view(request):
pass

in folder "users" create folder "templates", inside "users" folder, inside file layout.html

inside layout.html:

<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Users/title>
  </head>
  <body>
    {% block body %} {% endblock %}
  </body>
</html>

in folder "users" create folder "templates", inside "users" folder, inside login.html:
{{% extends "users/layout.html" %} {% block body %} {% if message %}

<div>{{ message }}</div>
{% endif %}
<h2>LOGIN</h2>
<form action="{% url 'login' %}" method="post">
  {% csrf_token %}
  <input type="text" name="username" , placeholder="Username" />
  <input type="password" name="password" , placeholder="Password" />
  <input type="submit" value="Login" />
</form>
{% endblock %}

in views.py changes about log in and log out:
from django.contrib.auth import authenticate, login, logout
from django.shortcuts import render
from django.http import HttpResponseRedirect
from django.urls import reverse
def index(request):
if not request.user.is_authenticated:
return HttpResponseRedirect(reverse('login'))
return render(request, "users/user.html" )
def login_view(request):
if request.method =="POST":
username = request.POST["username"]
password = request.POST['password']
user = authenticate(request, username=username, password=password)
if user is not None:
login(request, user)
return HttpResponseRedirect(reverse("index"))
else:
return render(request, "users/login.html", {
"message": "Invalid credentials."
})
return render(request, "users/login.html")
def logout_view(request):
logout(request)
return render(request, "users/login.html",{
"message": "Logged out."
})

and create user.html:
{% extends "users/layout.html" %} {% block body %}

<h2>Welcome, {{ request.user.first_name }}</h2>
<ul>
  <li>Username: {{request.user.username}}</li>
  <li>Email: {{request.user.email}}</li>
</ul>
<a href="{% url 'logout' %}">Log Out</a>
{% endblock %}

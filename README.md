# Django-Auth
Implementing social authentication in django using the famous django-allauth package

**I will be using [pipenv](https://pipenv.readthedocs.io/en/latest/) for my virtualenv stuff but you can use whatever package you want or completely skip a virtualenv (not recommended)**

```
pipenv install django django-allauth
pipenv shell
django-admin startproject mysite .
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver
```

Now for adding **django-allauth** features in the current django app we have to make 
some changes in project settings and urls:

**myproject/settings.py**
```
INSTALLED_APPS = [
	...
    'django.contrib.sites',		# <- needed for allauth

    # django-allauth
    'allauth',
    'allauth.account',
    'allauth.socialaccount',
    'allauth.socialaccount.providers.github',
]
```

And in the same file in the very bottom add

```
# django-allauth

AUTHENTICATION_BACKENDS = (
	# login by username backend from django
    "django.contrib.auth.backends.ModelBackend",

    # login by email backend by allauth
    "allauth.account.auth_backends.AuthenticationBackend",
)

# id of the site that is created by default by 'django.contrib.sites'
SITE_ID = 1

LOGIN_REDIRECT_URL = 'home'
```

After adding the following settings make changes in **myproject/urls.py** for allauth

```
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),

    # django-allauth urls
    path('accounts/', include('allauth.urls')),
]
```

Now ```migrate``` to update the database

```
python manage.py migrate
python manage.py runserver
```

Go to [127.0.0.1:8000/admin](http://127.0.0.1:8000/admin) and you will find the models 
created by ```django-allauth```

Go to [Sites](http://127.0.0.1:8000/admin/sites/site/) and edit the existing site by
changing the ```Domain name:``` to ```127.0.0.1:8000``` and leaving the ```Display name:``` to the default of ```example.com```

When you deploy your Django application live you’d replace ```127.0.0.1``` here and on 
**Github** with your actual production homepage. We use the localhost for testing 
purposes.

# Github OAuth
OAuth is an open standard for authentication between systems. When a user logs into our 
site with their Github account, we will redirect them to Github which then sends us a 
token that represents the user.

To configure a new OAuth application on Github, go to [https://github.com/settings/applications/new](https://github.com/settings/applications/new)

The Application name is what the user will see is requesting permission to access their 
Github account. The **Homepage URL** is as described. The **Authorization callback URL** takes a particular form for each integration as defined in the django-allauth [docs](http://django-allauth.readthedocs.io/en/latest/providers.html).

After hitting the “Register application” button you’ll be redirected to the page 
containing the **Client ID** and **Client Secret**. Also note that in the real-world, 
you’d never want to publicly reveal either of these keys!

Now run the server if not running and navigate to [Social Applications](http://127.0.0.1:8000/admin/socialaccount/socialapp/) and ```ADD SOCIAL APPLICATION```

We’re using **Github** so that’s the **Provider**. We add a **name** and then the 
**Client ID** and **Secret ID** from Github. Final step is to add our site to the 
**Chosen sites** on the bottom. Then click **save.**

# Home Page
Now if you remember that we added a variable in settings ```LOGIN_REDIRECT_URL``` and 
its value as ```'home'```. This is the url that django will redirect to after logging 
the user in after social login or the default login.

So let's create a simple class based view for home page

Create new file in **myproject** folder -> ```views.py```
```
from django.views.generic import TemplateView

class Home(TemplateView):
    template_name = 'home.html'
```

Since we gave a path for template, we also need to tell django about where the 
templates lies

```
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')],	# <- templates folder path
        'APP_DIRS': True,
        ...
```

Create a folder **templates** in the path where ```manage.py``` file is located and 
place the file ```home.html``` in it.

Checkout my ```home.html``` file from repo, I have used some bootstrap to add styling.

# Testing
Run the server and logout from admin site and navigate to home url [127.0.0.1:8000](http://127.0.0.1:8000/) and you will find a **Sign Up** button. Click that button and 
you will be redirected to sign up screen from github using your credentials. When 
authorized you will be redirected again to home url and you can see your username 
displayed on the home page.

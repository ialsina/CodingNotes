# Django manual

Manual created after the [Python Django Web Framework](https://www.youtube.com/watch?v=F5mRW0jo-U4) tutorial by [FreeCodeCamp](freecodecamp.org). The code can be found in [the course's repository](https://github.com/codingforentrepreneurs/Try-Django).

## Installation

### Creation of a virtual environment

In the directory of installation,

```
virtualenv -p python .
source bin/activate
```

If `python -V` reveals that the command `python` points to a version `2.*` of python, then use `python3` instead.

Other ways to achieve the same goal include:

```
virtualenv venv
virtualenv venv2 -p python3
virtualenv venv3 -p /usr/local/bin/python3
mkdir venv4 && cd venv4 && virtualenv . -p python3
python -m venv venv5 && source venv5/bin/activate
```
### Installation of Django in the environment

This (together with the activation) must be followed by

`pip install django`

The version used in the course is `django==2.0.7`

`pip freeze` shows the installation of the packages in the virtual environment.

## Creation of a blank Django project

```
mkdir src
django-admin startproject trydjango .
python manage.py runserver
```

## Settings

- `BASE_DIR`: the base directory
- `SECRET_KEY`
- `LLOWED_HOSTS`
- `INSTALLED_APPS`
- `MIDDLEWARE`
- `ROOT_URLCONF`
- `TEMPLATES`
- `WSGI_APPLICATION`
- `DATABASES`
- `AUTH_PASSWORD_VALIDATORS`
- `LANGUAGE_CODE`

## Built-in components

- To run the server: `python manage.py runserver`
- Create a superuser: `python manage.py createsuperuser`
- Use the superuser authentication to enter `localhost/admin/` where `localhost` is something like `127.0.0.1:8000` (see in the shell).

## A first App component

An App is a piece of functionality. Examples to create apps would include:
```
python manage.py startapp products
python manage.py startapp blog
python manage.py startapp profiles
python manage.py startapp cart
```

This creates several folders in the root directory (`src/`) with a directory for each one of those Apps. There, models can be implemented. For example, in `src/products/models.py`:

```
from django.db import models

class Product(models.Model):
    title = models.TextField()
    description = models.TextField()
    price = models.TextField()
```
After this, one needs to go back to `settings.py` and add `products` in the `INSTALLED_APPS` LIST. After this, it remains to run 
```
python manage.py makemigrations
python manage.py migrate
```
Now, if we want to change our model, and make our `src/products/models.py` look like
```
from django.db import models

class Product(models.Model):
    title = models.TextField()
    description = models.TextField()
    price = models.TextField()
    summary = models.TextField()
```
when, we run `python manage.py makemigrations` it will ask us to either
1. provide a one-off default value for the newly added field, or
2. quit, and add a default value for the newly added field.

We can quit, and let the file be

```
from django.db import models

class Product(models.Model):
    title = models.TextField()
    description = models.TextField()
    price = models.TextField()
    summary = models.TextField(default='A default value')
```
Then, run `python manage.py makemigrations` again.

Finally, we need to go to `src/products/admin.py` and import the model there:

```
from django.contrib import admin
from .models import Product
admin.site.register(Product)
```
Now, we can go to `localhost/admin/products/product` and add new products via the browser interface.

### Create product objects in the shell

We can run an InteractiveConsole with `python manage.py shell`:
```
>>> from products.models import Product
>>> Product.objects.all()
    <QuerySet[<Product: Product object (1)>]>
>>> Product.objects.create(title='New product 2', description='another one', price='20', summary='sweet')
>>> Product.objects.all()
    <QuerySet[<Product: Product object (1)>, <Product: Product object (2)>]>
```
They can now be accessed via the opened server.

### New model fields

For this change we will delete all the files in the `src/products/migrations/` folder, `__pycache__` and the `db.sqlite3` database file. Back in the `src/products/models.py` file, we will now implement more realistic fields, which can also be found in [Django's field types documentation](https://docs.djangoproject.com/en/2.0/ref/models/fields/#field-types)

```
from django.db import models

class Product(models.Model):
    title = models.CharField(max_length=120) # required arg
    description = models.TextField(blank=True, null=True)
    models.DecimalField(decimal_places=2, max_digits=1000)
    summary = models.TextField()
```

Then:
```
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser
```
If we go back to the web interface, we will see that the layout is different, as now the fields adapt to their content. On the other hand, if we use the shell instead, to add a product we would go
```
>>> Product.objects.create(title='New product 2', description='another one', price=20.99, summary='sweet')
```
where the price is a float, rather than a string.

### Changes to a model

Running `makemigrations` and `migrate` syncs the model with the database. If a change has been performed to the model, this will let the DB know about those changes, but questions arise, i.e. *what about hte previous things that were there*? For example, upon adding a new boolean (`BooleanField`) `featured`, there are several options:
1. Make the field nullable: `featured = models.BooleanField(null=True)`
2. Provide a default value for the field: `featured = models.BooleanField(default=True)`
3. Both: `featured = models.BooleanField(null=True, default=True)`
4. None, but upon running `makemigrations` provide a one-off default, e.g. `True`. 

Similarly, we can allow a field to be blank:
```
summary = models.TextField(blank=False, null=False)
```
This will show the field as bold in the web interface. The difference between blank and default is that blank refers to whether the field is rendered as required, whereas null is on a database level.

## Custom Homepage

To do this, we will create an app `pages`. In `src/pages/views.py`, we will write:

```
fromo django.http import HttpResponse
from django.shortcuts import render

def home_view(*args, **kwargs):
    return HttpResponse("<h1>Hello World</h1>")
```

As in `src/trydjango/settings.py` there is the entry `ROOT_URLCONF = 'trydjango.urls`, we know that the settings for URLs is in `src/trydjango/urls.py`. There, we need to add more paths:

```
from django.contrib import admin`
from django.urls import path`

from pages import views`

urlpatterns = [
    path('', views.home_view, name='home'),
    path('admin/', admin.site.urls),
]
```

This will work. However, a slightly better practice would involve writing:

```
from django.contrib import admin
from django.urls import path

from pages.views import home_view

urlpatterns = [
    path('', home_view, name='home')
    path('admin/', admin.site.urls),
]
```

Actually, whenever a request is made, there is a request being passed as an argument. There, we can see things like the user (`requests.user`). Thus, we can actually have the `src/pages/views.py` as:
```
from djagngo.http ipmort HttpResponse
from django.shortcuts ipmort render

def home_view(request, *args, **kwargs):
    return HttpResponse("<h1>Hello World</h1>")
```

## Django Templates

We first create a location for the templates: `src/templates`, and in there have a file `src/templates/home.html`:
```
<h1>Hello world</h1>
<p>This is a template.</p>
```
The part where we let Django know where the template is is missing. We need to go to `src/trydjango/settings.py` and add `os.path.join(BASE_DIR, "templates")` to that list.

We can now create a bunch of HTML files in `templates/`: `home.html`, `about.html`, `contact.html` and so on.

Now, in `src/pages/views`, we can change the `HttpResponse` by the `render` function:

```
from django.http import HttpResponse
from django.shortcuts import render

def home_view(request, *args, **kwargs):
    return render(request, "home.html", {})

def contact_view(request, *args, **kwargs):
    return render(request, "contact.html, {})

def about_view(request, *args, **kwargs):
    return render(request, "about.html, {})
```

provided that we also add these patterns in the `urlpatterns` variable of `src/trydjango/urls.py`.

### Templating Engine Basics

If in `home.html` we write:
```
<h1>Hello world</h1>
{{ request.user }}
<p>This is a template</p>
```
when we navigate into `localhost/home/`, we will see the username, if they are authenticated. If we write `request.user.is_authenticated` instead, we will see either `True` or `False`.

Our views are rendering out different HTML pages that might actually share some attributes, such as a navigation bar, or some metadata (`<title>` tag or `<descr>` tag, or `CSS` styles...). For that reason, we will create a `src/templates/base.html` file:
```
<!doctype html>
<html>
<head>
<title>Coding for Entrepreneurs is doing Try Django</title>
</head>
<body>
    <h1>This simulates a navbar</h1>
    {% block content %}{% endblock %}
</body>
</html>
```
Here, `block` and `endblock` is Django-specific syntax. `content` is a variable. Then, we can go to `home.html` and have:
```
{% extends 'base.html' %}

{% block content %}
    <h1>Hello world</h1>
    <p>This is a template</p>
{% endblock %}
```
This  is equivalent to `about.html`, `contact.html` and so on. Then, all these pages extend `base.html`, and specialize it with their unique content.

### Include Template Tag
Inheritance allows to reduce redundant code. The `include` template tag achieves that to a different level.

We can make a new file `src/templates/navbar.html`, which simulates a navbar.

```
<nav>
    <ul>
        <li>Brand</li>
        <li>Contact</li>
        <li>About</li>
    </ul>
</nav>
```
Then, we can go to our `src/templates/base.html` and let it be:
```
<!doctype html>
<html>
<head>
<title>Coding for Entrepreneurs is doing Try Django</title>
</head>
<body>
    {% include 'navbar.html' %}
    {% block content %}{% endblock %}
</body>
</html>
```

Once the project gets complicated, it gets easier to maintain. A slight change in the navbar can be done only in `navbar.html`.

### Rendering Context in a Template

The main purpose to use templates in Django is to render data that comes from the backend, from the database, i.e. changing the title of the page, or make content specific to a user. Django takes the template, some context, mashes it together, and returns raw HTML to the browser. Context could be any data type. For example, in the `about_view` (of `src/pages/views.py`).

```
def about_view(request, *args, **kwargs):
    my_context = {
        "my_text": "This is about us",
        "my_number": 123,
        "my_list": [123, 131, 312, "abc"],
    }
return render(request, "about.html", my_context)
```
Then, if we go to `src/templates/about.html`:
```
{% extends 'base.html' %}

{% block content %}
    <h1>About</h1>
    <p>This is a template</p>
<p>
{{ my_text }}, {{ my_number }}
</p>
{% endblock %}
```
This renders the context passed as a dictionary, whose keys become the context variables.

### For Loop in a Template

```
<ul>
{% for my_item in my_list %}
    <li> {{ forloop.counter }} - {{ my_item }}</li>
{% endfor %}
</ul>
```

### Conditions in a Template

```
<ul>
{% for my_item in my_list %}
    {% if my_item == 312 %}
        <li> {{ forloop.counter }} - {{ my_item|add:22 }}</li>
    {% elif my_item == "abc" %}
        <li>This is not the network</li>
    {% else %}
        <li>{{ forloop.counter }} - {{ my_item }}</li>
    {% endif %}
{% endfor %}
</ul>
```

The syntax `{{ my_item|add:22 }}` is called a [built-in template tag filter](https://docs.djangoproject.com/en/2.0/ref/templates/builtins/#built-in-filter-reference).

### Template tags and filters

There are many built-in template tags, such as [`block`](https://docs.djangoproject.com/en/2.0/ref/templates/builtins/#block), [`comment`](https://docs.djangoproject.com/en/2.0/ref/templates/builtins/#comment), [`cycle`](https://docs.djangoproject.com/en/2.0/ref/templates/builtins/#cycle), [`extends`](https://docs.djangoproject.com/en/2.0/ref/templates/builtins/#extends); and [`filters`](https://docs.djangoproject.com/en/2.0/ref/templates/builtins/#built-in-filter-reference), which are written with the pipe (`|`) operator (as the `|add` above). They can be stacked together by means of multiple pipe operators. Interesting filter operators are [`add`](https://docs.djangoproject.com/en/2.0/ref/templates/builtins/#add), [`safe`](https://docs.djangoproject.com/en/2.0/ref/templates/builtins/#safe), [`capfirst`](https://docs.djangoproject.com/en/2.0/ref/templates/builtins/#capfirst), [`upper`](https://docs.djangoproject.com/en/2.0/ref/templates/builtins/#upper), [`title`](https://docs.djangoproject.com/en/2.0/ref/templates/builtins/#title), [`escape`](https://docs.djangoproject.com/en/2.0/ref/templates/builtins/#escape), [`striptags`](https://docs.djangoproject.com/en/2.0/ref/templates/builtins/#striptags), [`slugify`](https://docs.djangoproject.com/en/2.0/ref/templates/builtins/#slugify).

## Render Data from the Database with a Model

In the terminal
```
python manage.py shell
>>> from products.models import Product
>>> obj = Product.objects.get(id=1)
>>> obj.title
```

In `src/products/views.py`:

```
from django.shortcuts.render
from .models import Product

def product_detail_view(request):
    obj = Product.objects.get(id=1)
    context = {
        'title': obj.title,
        'description': obj.description,
    }
    return render(request, "product/detail.html", context)
```

In `src/templates/product/detail.html`:
```
{% extends 'base.html' %}
{% block content %}
<h1>{{ title }}</h1>
<p>{% if description %}{{ description }}{% else %}Description Coming Soon{% endif %}</p>
{% endblock %}
```
In `src/trydjango/urls.py`:
```
from django.contrib import admin
from django.urls import path

from pages.views import home_view, contact_view, about_view
from products.views import product_detail_view

urlpatterns = [
    path('', home_view, name='home'),
    path('about/', about_view),
    path('contact/', contact_view),
    path('product/', product_detail_view),
    path('admin/', admin.site.urls),
]
```

We can also change our `src/products/views.py` to make the context be
```
def product_detail_view(request):
    obj = Product.objects.get(id=1)
    context = {'object': obj}
    return render(request, "product/detail.html", context)
```

Continuing, we would now change the `src/templates/product/detail.html`:
```
{% extends 'base.html' %}
{% block content %}
<h1>{{ object.title }}</h1>
<p>{% if object.description %}{{ object.description }}{% else %}Description Coming Soon{% endif %}</p>
{{ object.price }} 
{% endblock %}
```

### How Django Templates Load with Apps

When creating a Django app, the idea is to keep as much about the app inside the components directory. Thus, the app can be used in other projects. Let's create an in-app template. For that, create a file in the path `src/products/templates/products/product_detail.html`, and paste the content of the file `src/templates/product/detail.html`. If in `src/products/views.py` we call `render(request, "products/detail.html", context)`, that will raise an Error.

Looking at the traceback (Template-loader postmortem): First the `filesystem.Loader` is used (which comes from our settings, i.e. the `DIRS` list in `TEMPLATES` IN `settings.py`), which looks into `src/templates/products/detail.html`. Then, there are a few other  places (`lib/python3.6/.../admin/` and `lib/python3.6/.../auth`). Finally, Django falls back to our own app (`src/products/templates/products/detail.html`). This all fails, because none of these sources exist.

In `src/products/views.py`, we call instead `render(request, "products/product_detail.html", context)`. If we wanted to override this template, we could come to our `src/templates/` folder, and copy the path: `src/templates/products/product_detail.html`, and that one would be the one to show, instead of the in-app one. To not get confused, we will delete the whole `templates/products/` folder now.

## Model Forms

We would allow the users to save data into the database, obviously without using the `admin/` section or the shell. We will break down the basics to use the django model form.

In `src/products/forms.py`:

```
from django import forms
from .models import Product

class ProductForm(forms.ModelForm):
    class Meta:
        model = Product
        fields = [
            'title',
            'description',
            'price',
        ]
```

Now, in `src/products/views.py`:
```
from django.shortcuts import render
from .forms import ProductForm
from .models import Product

def product_create_view(request):
    form = ProductForm(request.POST or None)
    context = {'object': obj}
    return render (request, "products/product_create.html", context)

def product_detail_view(request):
    ... #as before
```

In `src/products/templates/products/product_create.html`:
```
{% extends 'base.html' %}
{% block content %}
<form method='POST'> {% csrf_token %}
{{ form.as_p }}
<input type='submit' value='Save' />
</form>
{% endblock %}
```

The line `{{ form.as_p }}` is a built-in method that turns the form into an HTML form rendered out with paragraph tags (`<p></p>`).

We now need to bring this into `src/trydjango/urls.py`, and make a new path, `path('create/', product_create_view)` (providing the right import). Make sure that everything is create, and one can navigate to `localhost/create/`.

Since the form doesn't feature the property `featured`, in `src/products/models.py`, we will change the line `featured` to
```
featured = models.BooleanField(default=False)
```
Then, run `makemigrations` and `migrate`.

This already works. Now to make the web form clear out after submit, we can make it re-render. To that end, in `src/products/views.py`, we will add a few lines to the function `product_create_view`:

```
def product_create_view(request):
    form = ProductForm(request.POST or None)
    if form.is_valid():
        form.save()
        form = ProductForm()
    context = {
        'form': form
    }
    return render(request, "products/product_create.html", context)
```

### Raw HTML Form

We will start from scratch HTML to build a form.

`src/products/templates/products/product_create.html`:

```
{% extends 'base.html' %}

{% block content %}

<form method='POST'>
    <input type='text' name='title' placeholder="your title"/>
    <input type='submit' value='Save' />
</form>

{% endblock %}
```

In `src/products/views`, we will create a `product_create_view`:

```
def product_create_view(request):
    context = {}
    return render(request, "products/product_create.html", context)
```

Right now if we go to `localhost/create/` and type in *abc*, hit *Save*, this returns an error: `CSRF token missing or incorrect`. If we change it to `method='GET'` instead, the same operation won't do anything, just change the URL. This is actually the behavior by default, so if no `method` is specified in the HTML form. This is similar to doing a search. We will specify the `POST` method again. The previous error has to do with security measures that Django has built in.

If we provide an action, i.e. `<form action='/search/' method='GET'>`, this takes us to a different URL `localhost/create/?title=abc`. `action` means to send it to a completely different url. An application of this would be to

```
{% extends 'base.html' %}

{% block content %}

<form action='http://www.google.com/search' method='GET'>
    <input type='text' name='title' placeholder="Your search"/>
    <input type='submit' value='Save' />
</form>

{% endblock %}
```

This would help us perform a google search. Doing the same with `POST` as a method yeilds a Google error. An option is to pass a dot as an action: `action='.'`. This brings us to whatever that URL is. 
Going back to the first version, and adding `{% csrf_token %}` will solve the original problem.

```
{% extends 'base.html' %}

{% block content %}

<form action='.' method='POST'>{% csrf_token %}
    <input type='text' name='title' placeholder="Your search"/>
    <input type='submit' value='Save' />
</form>

{% endblock %}
```

When we go to any URL, a `GET` request allows to retreive information. A `POST` request allows to store information. The backend can treat both kinds of data in a similar way. In the view, printing `request.GET` AND `request.POST`, will reveal that when navigating to `localhost/create/?title=abc`, a dictionary (actually `QueryDict`) with a key-value pair `{'title': ['abc']}` is passed in as a `GET` method. The item can be accessed via `request.GET.get('title')` or `request.GET['title']`. This is unsafe, so instead of doing that, we use `request.POST`, with the builtin middleware feature `crsf_token`. Equally, then the item can be grabbed from the QueryDict using `request.POST.get('title')` or `request.POST['title']`. In the view itself, this could be done to actually add a new instance of a product:
```
def product_create_view(request):
    if request.method == "POST":
        new_title = request.POST.get('title')
        Product.objects.create(title=new_title)
    context = {}
    return render(request, "products/product_create.html", context)
```
The condition will prevent the execution of the code if another method is used, but also when first loading the page or refreshing, as in both situations `request.POST.get('title')` would simple return `None`.

### Pure Django Form

In our `src/products/forms.py` we will create the following class:
```
class RawProductForm(forms.Form):
    title = forms.CharField()
    description = forms.CharField()
    price = forms.DecimalField()
```
 Notice that we are inheriting from `forms.Form`, instead of `forms.ModelForm`, and that there is no nested class `Meta`, also no information on the model. A description on the fields used can be found in the [Django documentation for Form fields](https://docs.djangoproject.com/en/2.0/ref/forms/fields).

Our file `src/products/views.py` would now have to look as follows:

```
from .forms import ProductForm, RawProductForm

from .models import Product

def product_create_view():
    form = RawProductForm()
    context = {
        "form": form,
    }
    return render(request, "products/product_create.html", context)1
```

And our file `src/products/templates/products/product_create.html` would look as:
```
{% extends 'base.html' %}

{% block content %}

<form action='.' method='POST'>{% csrf_token %}
    {{ form.as_p }}
    <input type='submit' value='Save' />
</form>

{% endblock %}
```

This already creates a form, but it won't save anything. An extra step is needed:
```
from .forms import ProductForm, RawProductForm

from .models import Product

def product_create_view():
    form = RawProductForm() # equivalent to RawProductForm(request.GET)
    if request.method == "POST":
        form = RawProductForm(request.POST)
        if form.is_valid():
            # it hasn't been "tampered" by the user via, say, Inspect
            print(form.cleaned_data)
            Product.objects.create(**form.cleaned_data)
        else:
            print(form.errors)
    context = {
        "form": form,
    }
    return render(request, "products/product_create.html", context)1
```

This (already only with the `RawProductForm(request.POST)` addition) makes Django render in the form a message of the kind "This field is required" as an HTML `<p></p>` tag.

### Form Widgets

```
class RawProductForm(forms.Form):
    title = forms.charField(
                        label='',
                        widget=forms.TextInput(
                                attrs={
                                    'placeholder": "Your title",
                                }
                            )
                        ) # required=True by default
    description = forms.CharField(
                        required=False,
                        widget=forms.Textarea(
                                attrs={
                                    "placeholder": "your description,
                                    "class": "new-class-name two",
                                    "id": "my-id-for-textarea",
                                    "rows": 20,
                                    "cols": 12,
                                }
                            )
                        )
    price = forms.DecimalField(initial=199.99)
```

The [core field arguments](https://docs.djangoproject.com/en/2.0/ref/forms/fields/#core-field-arguments) and the [widgets](https://docs.djangoproject.com/en/2.0/ref/forms/widgets/) can be reviewed in their respective documentations.

So far, what we have done is identical to the `ModelForm`, which renders out the same things as the `Form`. The only difference is how the view handles it.

### Form Validation Methods

Django has a lot of builtin validation for its fields. When we submit some data, Django will make sure that that data matches the builtin validati on. Now we are going to use the `ModelForm` again. It's really simple to override the field itself.

```
class ProductForm(forms.ModelForm):
    title = forms.charField(
                        label='',
                        widget=forms.TextInput(
                                attrs={
                                    'placeholder": "Your title",
                                }
                            )
    class Meta:
        model = Product
        fields = [
            'title',
            'description',
            'price',
        ]
```

Going back to `src/products/views.py`:

```
def product_create_view(request):
    form = ProductForm(request.POST or None)
    if form.is_valid():
        form.save()
        form = ProductForm()
    context = {'form': form}
    return render(request, "products/product_create.html", context)
```

Now, to implement a validation for title: for example, to assert it contains "CFE" and an email that ends with "edu".

```
class ProductForm(forms.ModelForm):
    title = forms.charField(
                        label='',
                        widget=forms.TextInput(
                                attrs={
                                    'placeholder": "Your title",
                                }
                            )
    email = forms.EmailField()
    class Meta:
        model = Product
        fields = [
            'title',
            'description',
            'price',
        ]

    def clean_title(self, *args, **kwargs):
        title = self.cleaned_data.get("title")
        if not "CFE" in title:
            raise forms.ValidationError("This is not a valid title")
        return title

    def clean_email(self, *args, **kwargs):
        email = self.cleaned_data.get("email")
        if email.endswidth("edu"):
        raise forms.ValidationError("This is not a valid email")
```

### Initial Values for Forms

To create initial data for forms, in `src/products/views.py`:

```
form django.shortcuts import render

from .forms import ProductForm, RawProductForm

from .models import Product

def render_initial_data(request):
    initial_data = {
        'title': "My awesome title"
    }
    obj = Product.objects.get(id=1)
    form = RawProductForm(request.POST or None, initial=initial_data)
    if forms.is_valid():
        form.save()
    context = {
        'form': form
    }
    return render(request, "products/product_create.html", context)
```

## Dynamic URL Routing

This consists in changing the content based on the URL.

In `src/products/views.py`:

```
from django.shortcuts import render
from .models import Product

def dynamic_lookup_view(request, my_id):
    obj = Product.objects.get(id=my_id)
    context = {
        "object": obj
    }
```

In `src/urls.py`, set
```
urlpatterns = [
    path('products/<int:my_id>/', dynamic_lookup_view, name='product')
```
The part in `<>` is passed to the view as an argument. Another possibility would be to put as an argument `<slug:slug>`.

## Handle `DoesNotExist`

In `src/products/views.py`:

```
from django.shortcuts import render, get_object_or_404
from .models import Product

def dynamic_lookup_view(request, my_id):
    obj = get_object_or_404(Product, id=my_id
    context = {
          "object": obj
    }
    return render(request, "products/product_detail.html", context)
```

Or else,

```
from django.http import Http404
from django.shortcuts import render, get_object_or_404
from .models import Product

def dynamic_lookup_view(request, my_id):
    try:
        obj = Product.object.get(id=my_id) 
    except Product.DoesNotExist:
        raise Http404
    context = {
        "object": obj
    }
    return render(request, "products/product_detail.html", context)
```

## Delete and Confirm

To delete an object we need `obj.delete()`. However, this happens on a `GET` request, but we would rather a `POST` request.

We will first create a form that does this request for us, in `src/products/templates/products/product_delete.html`:

```
{% extends 'base.html' %}

{% block content %}

<form action='.' method='POST'>{% csrf_token %}
    <h1>Do you want to delete the product "{{ object.title }}"?</h1>
    <p><input type='submit' value'Yes' /> <a href ='../'>Cancel</a></p>
</form> 

{% endblock %}
```

The `urlpatterns` in `src/urls.py`:
```
urlpatterns = [
    path('products/<int:my_id>/delete/', product_delete_view, name='product-delete')
```

Finally, in `src/products/views.py`:

```
from django.shortcuts import render, get_object_or_404, redirect
from .models import Product

def product_delete_view(request, my_id):
    obj = get_object_or_404(Product, id=my_id)
    # POST request
    if request.method == "POST":
        # confirming delete
        obj.delete()
        # Redirect so as not to see the 404 error
        return redirect('../../')
    context = {
        "object": obj
    }
    return render(request, "products/product_delete.html", context)
```

## View of a List of Database Objects

To list out objects we first have to get a `QuerySet`.

In `src/products/views.py`:

```
from django.shortcuts import render, get_object_or_404, redirect
from .models import Product

def product_list_view(request):
    queryset = Product.objects.all() # list of objects
    context = {
        "object_list": queryset
    }
    return render(request, "products/product_list.html", context)
```

In `src/products/templates/products/product_list.html`:

```
{% extends 'base.html' %}

{% block content %}

{{ object_list }}

{% for instance in object_list %}
    <p>{{ instance.id }} - {{ instance.title }}</p>

{% endfor %}

{% endblock %}
```

## Dynamic Linking of URLs

To create a link to the detail of any given product, we might do this:

```
{% extends 'base.html' %}

{% block content %}

{{ object_list }}

{% for instance in object_list %}
    <p>{{ instance.id }} - <a href ='/products/{{ instance.id }}/'>{{ instance.title }}</a></p>

{% endfor %}

{% endblock %}
```

And this would work. The problem is that if we ever changed the URL structure, say `/product/` to `/p/`, we would have to go back and change everything everywhere (`urls.py` and so on). An alternative is to create an *instance method* on our model. In `src/products/models.py`:

```
from django.db import models
from django.urls import reverse

from django.db import models

class Product(models.Model):
    title = models.CharField(max_length=120)
    description = models.TextField(blank=True, null=True)
    models.DecimalField(decimal_places=2, max_digits=1000)
    summary = models.TextField(blank=False, null=False)
    featured = models.BooleanField(default=False)

    def get_absolute_url(self):
        return f"/products/{self.id}/"
```

Then, in `src/products/templates/products/product_list.html" we would do instead:

```
{% extends 'base.html' %}

{% block content %}

{{ object_list }}

{% for instance in object_list %}
    <p>{{ instance.id }} - <a href ='{{ instance.get_absolute_url }}'>{{ instance.title }}</a></p>

{% endfor %}

{% endblock %}
```

Then, we would know that everywhere that `get_absolute_url` works, this would update as well.


## Django URLs Reverse

It's time to transition our `get_absolute_url` to be dynamic as well. If we look at out 'src/trydjango/urls.py`, it looks like this:

```
from django.contrib import admin
from django.urls import path

from pages.views import home_view, contact_view, about_view
from products.views import (
    product_details_view,
    product_delete_view,
    product_create_view,
    product_list_view,
    render_initial_data,
    dynamic_lookup_view
    )

urlpatterns = [
    path('products/', product_list_view, name='product-list'),
    path('products/<int:my_id>/', dynamic_lookup_view, name='product-detail'),
    path('products/<int:my_id>/delete/', product_delete_view, name='product-delete'),
]
```

We named our URLs on purpose. If we go to `src/products/models.py` and change it to

```
from django.db import models
from django.urls import reverse

from django.db import models

class Product(models.Model):
    title = models.CharField(max_length=120)
    description = models.TextField(blank=True, null=True)
    models.DecimalField(decimal_places=2, max_digits=1000)
    summary = models.TextField(blank=False, null=False)
    featured = models.BooleanField(default=False)

    def get_absolute_url(self):
        return reverse("product-detail", kwargs={"my_id": self.id})
```

This is a nice and clean way to make sure that our URLs are dynamic, i.e. in my `urls.py`, if we ever changed those `products/` to `p/` those links would actually update. It does it accross the site.

## In App URLs and and Namespacing

At this stage, if everything is cleaned up into their own views with names that match, we would end up with a file `src/products/views.py` file that contains the functions `product_create_view`, `product_update_view`, `product_list_view, product_detail_view`, `product_delete_view`. The file `src/trydjango/urls.py` would look as follows:

```
from django.contrib import admin
from django.urls import path

from pages.views import home_view, contact_view, about_view
from products.views import (
    product_details_view,
    product_delete_view,
    product_create_view,
    product_list_view,
    render_initial_data,
    dynamic_lookup_view
    )

urlpatterns = [
    path('products/', product_list_view, name='product-list'),
    path('products/create/', product_create_view, name='product-list'),
    path('products/<int:my_id>/', product_detail_view, name='product-detail'),
    path('products/<int:my_id>/update', product_update_view, name='product-update'),
    path('products/<int:my_id>/delete/', product_delete_view, name='product-delete'),

    path('', home_view, name='home'),
    path('about/', about_view, name='product-detail'),
    path('contact/', contact_view),
    path('admin/', admin.site.urls),
]
```
However, this is not that reusable as an app, as we have go import all of these views. Besides that, a problem would arise if the same `name` and keyword arguments (i.e. `<int:id>`) somewhere else (e.g. change 'about/' by 'about/<int:my_id>/. The model is based off of that, and it produces unintended behavior.

To solve this, we can create a new module inside of the up, i.e. `src/products/urls.py`:

```
from django.urls import path

from .views import (
    product_details_view,
    product_delete_view,
    product_create_view,
    product_list_view,
    render_initial_data,
    dynamic_lookup_view
    )

app_name = 'products'
urlpatterns = [
    path('', product_list_view, name='product-list'),
    path('create/', product_create_view, name='product-list'),
    path('<int:my_id>/', product_detail_view, name='product-detail'),
    path('<int:my_id>/update', product_update_view, name='product-update'),
    path('<int:my_id>/delete/', product_delete_view, name='product-delete'),
]

```

where we got rid of the `products/` preffix in the URLs, and `app_name` is to ensure a namespace. This calls for a further change in the model. Thus, in `src/products/models.py`, the model incorporates such namespace. It now looks:

```
from django.db import models
from django.urls import reverse

from django.db import models

class Product(models.Model):
    title = models.CharField(max_length=120)
    description = models.TextField(blank=True, null=True)
    models.DecimalField(decimal_places=2, max_digits=1000)
    summary = models.TextField(blank=False, null=False)
    featured = models.BooleanField(default=False)

    def get_absolute_url(self):
        return reverse("products:product-detail", kwargs={"my_id": self.id})
```

Then, back in `src/trydjango/urls.py`:

```
from django.contrib import admin
from django.urls import include, path

from pages.views import home_view, contact_view, about_view

urlpatterns = [
    path('products/' include('products.urls')),
    path('', home_view, name='home'),
    path('about/', about_view, name='product-detail'),
    path('contact/', contact_view),
    path('admin/', admin.site.urls),
]
```

It is thus important to realise that having names in the `urls.py` doesn't mean that there are any checks to make sure that those names are unique. Doing this, we could change our `app_name` if needed, which would allow our namespace to change and therefore our `reverse` method to change as well. This is why `reverse` is important: it allows for portability.


## Class Based Views

At this stage, there is the following suggestion:

1. Create a new App named `Blog`
2. Add `Blog` to the Django project
3. Create a `Model` named `Article`
4. Run `migrations`
5. Create a `ModelForm` for `Article`
6. Create `article_list.html` and `article_detail.html` Template
7. Add `Article` `Model` to the Admin
8. Save a new `Articl` object in the admin

Confused? Start [here](https://kirr.co/9ypik6).

### ListView

In `src/blog/views.py`:

```
from django.shortcuts import render

from django.views.generic import CreateView, DetailView, ListView, UpdateView, DeleteView
from .models import Article

class ArticleListView(ListView):
    template_name = 'articles/article_list.html'
    queryset = Article.objects.all()
```

What we just did is overriding the default template it looks up. without the clsas attribute `template_name`, it was looking for `blog/article_list.html`, i.e. the generic one.

In `src/blog/urls.py`:
```
from django.urls import path
from .views import ArticleListView

app_name = 'articles'
urlpatterns = [
    path('', ArticleListView.as_view(), name='article-list'),
]
```

Here, the `as_view()` method turns that into a function-based view.

Our `src/trydjango/urls.py` is now:
```
from django.contrib import admin
from django.urls import include, path

from pages.views import home_view, contact_view, about_view

urlpatterns = [
    path('blog/' include('blog.urls')),
    path('products/' include('products.urls')),
    path('', home_view, name='home'),
    path('about/', about_view, name='product-detail'),
    path('contact/', contact_view),
    path('admin/', admin.site.urls),
]
```

### DetailView

In `src/blog/views.py`:

```
from django.shortcuts import render, get_object_or_404

from django.views.generic import CreateView, DetailView, ListView, UpdateView, DeleteView
from .models import Article

class ArticleListView(ListView):
    template_name = 'articles/article_list.html'
    queryset = Article.objects.all()

class ArticleDetailView(DetailView):
    template_name = 'articles/article_detail.html'
    queryset = Article.objects.all()
```


In `src/blog/urls.py`:
```
from django.urls import path
from .views import (
    ArticleDetailView,
    ArticleListView,
)

app_name = 'articles'
urlpatterns = [
    path('', ArticleListView.as_view(), name='article-list'),
    path('<int:pk>/' ArticleDetailView.as_view(), name='article-detail'),
]
```

#### An alternative

Instead of using `pk` (i.e. *primary key*), we can use a custom lookup keyword, `id`. In `src/blog/urls.py`:
```
from django.urls import path
from .views import (
    ArticleDetailView,
    ArticleListView,
)

app_name = 'articles'
urlpatterns = [
    path('', ArticleListView.as_view(), name='article-list'),
    path('<int:id>/' ArticleDetailView.as_view(), name='article-detail'),
]
```

At first, this would yield an error:

> `Generic detail view ArticleDetailView must be called with either an object pk or a slug.`

Those correlate to different fields in the model (i.e. `Article`). `pk` stands for *primary key*, and there it is actually the same as `id`. If we use `id` instead, we have to override a method in our views. In `src/blog/views.py`:

```
from django.shortcuts import render, get_object_or_404

from django.views.generic import CreateView, DetailView, ListView, UpdateView, DeleteView
from .models import Article

class ArticleListView(ListView):
    template_name = 'articles/article_list.html'
    queryset = Article.objects.all()

class ArticleDetailView(DetailView):
    template_name = 'articles/article_detail.html'
    # queryset = Article.objects.all() # we don't need this anymore
    def get_object(self):
        id_ = self.kwargs.get("id")
        return get_object_or_404(Article, id_)
```

Class-based views inherit from different pieces, and in the case of `DetailView`, `get_object` is a prmiary method to override. When doing this `queryset` limits the choices available for the `DetailView`. An example would be: `queryset = Article.objects.filter(id__gt=1)`, which would limit the queryset to articles whose `id` is greater than 1. However, this would again need to go together with `<int:pk>` as the path lookup in `src/blog/urls.py`. 



### CreateView and UpdateView

In `src/blog/views.py`:

```
from django.shortcuts import render, get_object_or_404

from django.views.generic import CreateView, DetailView, ListView, UpdateView, DeleteView
from .models import Article
from .forms import ArticleModelForm

class ArticleCreateView(CreateView):
    template_name = 'articles/article_create.html'
    form_class = ArticleModelForm
    queryset = Article.objects.all()
    # success_url = <whatever>

    def form_valid(self, form):
        print(form.cleaned_data)
        return super().form_valid(form)

    # def get_success_url(self):
    #    return <whatever> #either method or class attribute

class ArticleListView(ListView):
    template_name = 'articles/article_list.html'
    queryset = Article.objects.all()

class ArticleDetailView(DetailView):
    template_name = 'articles/article_detail.html'
    # queryset = Article.objects.all() # we don't need this anymore
    def get_object(self):
        id_ = self.kwargs.get("id")
        return get_object_or_404(Article, id_)

class ArticleUpdateView(UpdateView):
    template_name = 'articles/article_create.html'
    form_class = ArticleModelForm
    queryset = Article.objects.all()

    def get_object(self0:
        id_ = self.kwargs.get("id")
        return get_object_or_404(Article, id=id_)

    def form_valid(self, form):
        form.cleaned_data
        return super().form_valid(form)
```

In `src/blogs/forms.py`:
```
from django import forms
form .models import Article

class ArticleModelForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = [
            'title',
            'content',
            'active',
        ]
    def clean_title(self):
        ...
```

In `src/blog/models.py`:
```
from django.db import models

class Article(models.Model):
    title = models.CharField(max_length=120)
    content = models.TextField()
    active = models.BooleanField(default=True)

    def get_absolute_url(self):
         reverse("blog:article-detail", kwargs={"id": self.id})
```

In `src/blog/urls.py`:
```
from django.urls import path
from .views import (
    ArticleCreateView
    ArticleDetailView,
    ArticleListView,
)

app_name = 'articles'
urlpatterns = [
    path('', ArticleListView.as_view(), name='article-list'),
    path('create/', ArticleCreateView.as_view(), name='article-create'),
    path('<int:id>/' ArticleDetailView.as_view(), name='article-detail'),
    path('<int:id>/update' ArticleUpdateView.as_view(), name='article-update'),
]
```

### DeleteView

In `src/blog/views.py`:

```
from django.shortcuts import render, get_object_or_404
from django.urls import reverse

from django.views.generic import CreateView, DetailView, ListView, UpdateView, DeleteView
from .models import Article
from .forms import ArticleModelForm

class ArticleCreateView(CreateView):
    template_name = 'articles/article_create.html'
    form_class = ArticleModelForm
    queryset = Article.objects.all()
    # success_url = <whatever>

    def form_valid(self, form):
        print(form.cleaned_data)
        return super().form_valid(form)

    # def get_success_url(self):
    #    return <whatever> #either method or class attribute

class ArticleListView(ListView):
    template_name = 'articles/article_list.html'
    queryset = Article.objects.all()

class ArticleDetailView(DetailView):
    template_name = 'articles/article_detail.html'
    # queryset = Article.objects.all() # we don't need this anymore
    def get_object(self):
        id_ = self.kwargs.get("id")
        return get_object_or_404(Article, id_)

class ArticleUpdateView(UpdateView):
    template_name = 'articles/article_create.html'
    form_class = ArticleModelForm
    queryset = Article.objects.all()

    def get_object(self0:
        id_ = self.kwargs.get("id")
        return get_object_or_404(Article, id=id_)

    def form_valid(self, form):
        form.cleaned_data
        return super().form_valid(form)

class ArticleDeleteView(DeleteView):
    template_name = 'articles/article_delete.html'

    def get_object(self0:
        id_ = self.kwargs.get("id")
        return get_object_or_404(Article, id=id_)

    def get_success_url(self):
         reverse('articles:article-list')
```

Not having a `get_success_url` method yields an error. That error prevents the deletion, unlike the creation. 

In `src/blog/urls.py`:
```
from django.urls import path
from .views import (
    ArticleCreateView
    ArticleDetailView,
    ArticleListView,
)

app_name = 'articles'
urlpatterns = [
    path('', ArticleListView.as_view(), name='article-list'),
    path('create/', ArticleCreateView.as_view(), name='article-create'),
    path('<int:id>/' ArticleDetailView.as_view(), name='article-detail'),
    path('<int:id>/update' ArticleUpdateView.as_view(), name='article-update'),
    path('<int:id>/delete' ArticleDeleteView.as_view(), name='article-delete'),
]
```

## Function Based View to Class Based View

This is based around a new app `courses` (`src/courses/`). A function-based view would look as:

```
from django.shortcuts import render
def my_fbv(request, *args, **kwargs):
    return render(reques;t, 'about.html', {})
```

To convert this into a class-based view, we will inherit from a class named view. The name, as a function-based view, doesn't really matter... but as a class-based view, does: `get`/`post`.

```
from django.shortcuts import render
from django.views import View

class CourseView(View):
    def get(self, request, *args, **kwargs):
        # GET method
        return render(request, "about.html", {})
```

In `courses/urls.py`:

```
from django.urls import path
from .viefws import (
    CourseView,
)

app_name = 'courses'
urlpatterns = [
    path('', CourseView.as_view(), name='courses-list'),
]
```

An advantage of Class-based views is that they are flexible:

```
from django.shortcuts import render
from django.views import View

class CourseView(View):
template_name = "about.html"
    def get(self, request, *args, **kwargs):
        # GET method
        return render(request, self.template_name, {})
```
Also, now in `src/courses/urls.py`, we could make the call passing an argument as the template:
> `path(''CourseView.as_view(template='contact.html', name='courses-list')`.

### Raw Detail Class Based View

We will now bring the model into the view to have a `DetailView`.

In `courses/urls.py`:

```
from django.urls import path
from .viefws import (
    CourseView,
)

app_name = 'courses'
urlpatterns = [
    path('', CourseView.as_view(template_name='contact.html' ), name='courses-list'),
    path('<int:id>/', CourseView.as_view(), name='courses-detail'),
]
```

Now, if there is no `id` in the URL, the `template_name` will be overridden in the view.

We will create the file `src/courses/templates/courses/course_detail.html`, with some basic HTML.


In `courses/views.py`:
```
from django.shortcuts import render, get_object_or_404
from django.views import View

class CourseView(View):
    template_name = "courses/course_detail.html"
    def get(self, request, id=None, *args, **kwargs):
        # GET method
        context = {}
        if id is not None:
            obj = get_object_or_404(Course, id=id)
            context['object'] = obj
        return render(request, self.template_name, context)
```

### Raw List Class Based View

In `courses/views.py`:
```
from django.shortcuts import render, get_object_or_404
from django.views import View

class CourseListView(View):
    template_name = "courses/course_list.html"
    queryset = Course.object.all()

    def get(self, request, *args, **kwargs):
        context = {'object_list': self.queryset}
        return render(request, self.template_name, context)

class CourseView(View):
    template_name = "courses/course_detail.html"
    def get(self, request, id=None, *args, **kwargs):
        # GET method
        context = {}
        if id is not None:
            obj = get_object_or_404(Course, id=id)
            context['object'] = obj
        return render(request, self.template_name, context)
```

In `courses/urls.py`:

```
from django.urls import path
from .viefws import (
    CourseView,
    CourseListView
)

app_name = 'courses'
urlpatterns = [
    path('', CourseListView.as_view(), name='courses-list'),
    path('<int:id>/', CourseView.as_view(), name='courses-detail'),
]
```

As an extra step, we will now change the `CourseListView` to be:

```
class CourseListView(View):
    template_name = "courses/course_list.html"
    queryset = Course.object.all()

    def get_queryset(self):
        return self.queryset

    def get(self, request, *args, **kwargs):
        context = {'object_list': self.get_queryset()}
        return render(request, self.template_name, context)
```

which makes sense if we need to inherit from it:

```
class ChildListView(CourseListView):
    queryset = Course.objects.filter(id=1)
```
 
### Raw Create Class Based View

In `courses/views.py`:
```
from django.shortcuts import render, get_object_or_404
from django.views import View

from .forms import CourseModelForm
from .models import Course

class CourseCreateView(View):
    template_name = "courses/course_create.html"
    queryset = Course.object.all()

    def get(self, request, *args, **kwargs):
        # GET method
        form = CourseModelForm()
        context = {"form": form}
        return render(request, self.template_name, context)

    def post(self, request, *args, **kwargs):
        # POST method
        form = CourseModelForm(request.POST)
        if form.is_valid():
            form.save()
            form = CourseModelForm() # to reinitialize the actual form once submitted
        context = {"form": form}
        return render(request, self.template_name, context)

class CourseListView(View):
    template_name = "courses/course_list.html"
    queryset = Course.object.all()

    def get_queryset(self):
        return self.queryset

    def get(self, request, *args, **kwargs):
        context = {'object_list': self.get_queryset()}
        return render(request, self.template_name, context)

class CourseView(View):
    template_name = "courses/course_detail.html"
    def get(self, request, id=None, *args, **kwargs):
        # GET method
        context = {}
        if id is not None:
            obj = get_object_or_404(Course, id=id)
            context['object'] = obj
        return render(request, self.template_name, context)
```

In `courses/urls.py`:

```
from django.urls import path
from .viefws import (
    CourseView,
    CourseListView,
    CourseCreateView,
)

app_name = 'courses'
urlpatterns = [
    path('', CourseListView.as_view(), name='courses-list'),
    path('create/', CourseCreateView.as_view(), name='courses-create'),
    path('<int:id>/', CourseView.as_view(), name='courses-detail'),
]
```

### Raw Validation on a Post Method

In `src/courses/forms.py`:

```
from django import forms

from .models import Course

class CourseModelForm(forms.ModelForm):
    class Meta:
        model = Course
        fields = [
            'title,
        ]
    def clean_title(self):
        title = self.cleaned_data.get('title')
        if title.lower() == 'abc':
            raise forms.ValidationError("This is not a valid title")
        return title
```

### Raw Update Class Based View

In `courses/views.py`:
```
from django.shortcuts import render, get_object_or_404
from django.views import View

from .forms import CourseModelForm
from .models import Course

class CourseUpdateView(View):
    template_name = "courses/course_update.html"
    queryset = Course.object.all()

    def get_oboject(self):
        id = self.kwargs.get('id')
        obj = None
        if id is not None:
            obj = get_object_or_404(Course, id=id)
        return obj

    def get(self, request, id=None, *args, **kwargs):
        # GET method
        context = {}
        obj = self.get_object()
        if obj is not None:
            form = CourseModelForm(instance=obj)
            context['object'] = obj
            context['form'] = form
        context = {"form": form}
        return render(request, self.template_name, context)

    def post(self, request, id=None, *args, **kwargs):
        # POST method
        context = {}
        obj = self.get_object()
        if obj is not None:
            form = CourseModelForm(request.POST, instance=obj  )
            if form.is_valid():
                form.save()
            context['object'] = obj
            context['form'] = form
        context = {"form": form}
        return render(request, self.template_name, context)

class CourseCreateView(View):
    template_name = "courses/course_create.html"
    queryset = Course.object.all()

    def get(self, request, *args, **kwargs):
        # GET method
        form = CourseModelForm()
        context = {"form": form}
        return render(request, self.template_name, context)

    def post(self, request, *args, **kwargs):
        # POST method
        form = CourseModelForm(request.POST)
        if form.is_valid():
            form.save()
            form = CourseModelForm() # to reinitialize the actual form once submitted
        context = {"form": form}
        return render(request, self.template_name, context)

class CourseListView(View):
    template_name = "courses/course_list.html"
    queryset = Course.object.all()

    def get_queryset(self):
        return self.queryset

    def get(self, request, *args, **kwargs):
        context = {'object_list': self.get_queryset()}
        return render(request, self.template_name, context)

class CourseView(View):
    template_name = "courses/course_detail.html"
    def get(self, request, id=None, *args, **kwargs):
        # GET method
        context = {}
        if id is not None:
            obj = get_object_or_404(Course, id=id)
            context['object'] = obj
        return render(request, self.template_name, context)
```

### Raw Delete Class Based View

In `courses/views.py`:
```
from django.shortcuts import render, get_object_or_404, redirect
from django.views import View

from .forms import CourseModelForm
from .models import Course

class CourseUpdateView(View):
    template_name = "courses/course_update.html"
    queryset = Course.object.all()

    def get_oboject(self):
        id = self.kwargs.get('id')
        obj = None
        if id is not None:
            obj = get_object_or_404(Course, id=id)
        return obj

    def get(self, request, id=None, *args, **kwargs):
        # GET method
        context = {}
        obj = self.get_object()
        if obj is not None:
            form = CourseModelForm(instance=obj)
            context['object'] = obj
            context['form'] = form
        context = {"form": form}
        return render(request, self.template_name, context)

    def post(self, request, id=None, *args, **kwargs):
        # POST method
        context = {}
        obj = self.get_object()
        if obj is not None:
            form = CourseModelForm(request.POST, instance=obj  )
            if form.is_valid():
                form.save()
            context['object'] = obj
            context['form'] = form
        context = {"form": form}
        return render(request, self.template_name, context)

class CourseDeleteView(View):
    template_name = "courses/course_update.html"

    def get_oboject(self):
        id = self.kwargs.get('id')
        obj = None
        if id is not None:
            obj = get_object_or_404(Course, id=id)
        return obj

    def get(self, request, id=None, *args, **kwargs):
        # GET method
        context = {}
        obj = self.get_object()
        if obj is not None:
            context['object'] = obj
        context = {"form": form}
        return render(request, self.template_name, context)

    def post(self, request, id=None, *args, **kwargs):
        # POST method
        context = {}
        obj = self.get_object()
        if obj is not None:
            obj.delete()
            context['object'] = obj
            return redirect('/courses/')
        return render(request, self.template_name, context)

class CourseCreateView(View):
    template_name = "courses/course_create.html"
    queryset = Course.object.all()

    def get(self, request, *args, **kwargs):
        # GET method
        form = CourseModelForm()
        context = {"form": form}
        return render(request, self.template_name, context)

    def post(self, request, *args, **kwargs):
        # POST method
        form = CourseModelForm(request.POST)
        if form.is_valid():
            form.save()
            form = CourseModelForm() # to reinitialize the actual form once submitted
        context = {"form": form}
        return render(request, self.template_name, context)

class CourseListView(View):
    template_name = "courses/course_list.html"
    queryset = Course.object.all()

    def get_queryset(self):
        return self.queryset

    def get(self, request, *args, **kwargs):
        context = {'object_list': self.get_queryset()}
        return render(request, self.template_name, context)

class CourseView(View):
    template_name = "courses/course_detail.html"
    def get(self, request, id=None, *args, **kwargs):
        # GET method
        context = {}
        if id is not None:
            obj = get_object_or_404(Course, id=id)
            context['object'] = obj
        return render(request, self.template_name, context)
```

In `courses/urls.py`:

```
from django.urls import path
from .viefws import (
    CourseView,
    CourseListView,
    CourseCreateView,
)

app_name = 'courses'
urlpatterns = [
    path('', CourseListView.as_view(), name='courses-list'),
    path('create/', CourseCreateView.as_view(), name='courses-create'),
    path('<int:id>/', CourseView.as_view(), name='courses-detail'),
    path('<int:id>/update/', CourseUpdateView.as_view(), name='courses-update'),
    path('<int:id>/delete/', CourseDeleteView.as_view(), name='courses-delete'),
]
```

### Custom Mixin for Class Based Views

`Mixin`s are part of the reason why Class-based views are so good. A `Mixin` allows us to extend a Class-based view with some new code, helping us reduce redundancy in our code.

In `courses/views.py`:
```
from django.shortcuts import render, get_object_or_404, redirect
from django.views import View

from .forms import CourseModelForm
from .models import Course

class CourseObjectMixin(object):
    model = None
    # lookup = 'id'

    def get_object(self):
        id = self.kwargs.get('id')
        obj = None
        if id is not None:
            obj = get_object_or_404(self.model, id=id)
        return obj

class CourseUpdateView(CourseObjectMixin, View):
    template_name = "courses/course_update.html"
    queryset = Course.object.all()

    def get_oboject(self):
        id = self.kwargs.get('id')
        obj = None
        if id is not None:
            obj = get_object_or_404(Course, id=id)
        return obj

    def get(self, request, id=None, *args, **kwargs):
        # GET method
        context = {}
        obj = self.get_object()
        if obj is not None:
            form = CourseModelForm(instance=obj)
            context['object'] = obj
            context['form'] = form
        context = {"form": form}
        return render(request, self.template_name, context)

    def post(self, request, id=None, *args, **kwargs):
        # POST method
        context = {}
        obj = self.get_object()
        if obj is not None:
            form = CourseModelForm(request.POST, instance=obj  )
            if form.is_valid():
                form.save()
            context['object'] = obj
            context['form'] = form
        context = {"form": form}
        return render(request, self.template_name, context)

class CourseDeleteView(CourseObjectMixin, View):
    template_name = "courses/course_update.html"

    def get(self, request, id=None, *args, **kwargs):
        # GET method
        context = {}
        obj = self.get_object()
        if obj is not None:
            context['object'] = obj
        context = {"form": form}
        return render(request, self.template_name, context)

    def post(self, request, id=None, *args, **kwargs):
        # POST method
        context = {}
        obj = self.get_object()
        if obj is not None:
            obj.delete()
            context['object'] = obj
            return redirect('/courses/')
        return render(request, self.template_name, context)

class CourseCreateView(View):
    template_name = "courses/course_create.html"
    queryset = Course.object.all()

    def get(self, request, *args, **kwargs):
        # GET method
        form = CourseModelForm()
        context = {"form": form}
        return render(request, self.template_name, context)

    def post(self, request, *args, **kwargs):
        # POST method
        form = CourseModelForm(request.POST)
        if form.is_valid():
            form.save()
            form = CourseModelForm() # to reinitialize the actual form once submitted
        context = {"form": form}
        return render(request, self.template_name, context)

class CourseListView(View):
    template_name = "courses/course_list.html"
    queryset = Course.object.all()

    def get_queryset(self):
        return self.queryset

    def get(self, request, *args, **kwargs):
        context = {'object_list': self.get_queryset()}
        return render(request, self.template_name, context)

class CourseView(CourseObjectMixin, View):
    template_name = "courses/course_detail.html"
    def get(self, request, id=None, *args, **kwargs):
        # GET method
        context = {'object': self.get_object()}
        if id is not None:
            context['object'] = obj
        return render(request, self.template_name, context)
```

In `CourseObjectMixin`, if the attribute `lookup` is used, then everywhere `id` could be replaced by `lookup`, and `'id'` by `self.lookup`, and we would have a generic way to perform lookups. This would call for a potential overriding of the attribute `lookup` in the classes that inherit from `CourseObjectMixin`.



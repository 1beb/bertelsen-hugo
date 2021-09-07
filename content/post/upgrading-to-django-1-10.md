---
author: "Brandon Erik Bertelsen"
title: "Updating Django 1.4 to 1.10 Learnings"
date: "2016-08-15"
categories:
- Python
- Django
---

I had a project that I wrote in Django, 1.4, back in 2012. It's still in production and there are no problems with it. The only exception being that there are planned changes to the database schema. Weighing my options, I decided to move forward with an upgrade to Django 1.10, because migrations are now "built-in" and data reliability is more important to me than a few extra hours of trouble-shooting the differences between the versions.

Here are the issues that effected me, when moving from Django 1.4 to 1.10.

### Changes to settings.py

There are no longer `TEMPLATE_*` variables in the `settings.py` file you should remove them

- The settings file now uses os and you should setup a variable called BASE_DIR

```python
import os  
BASE_DIR = "/path/to/your/app"  
```

- The TEMPLATES_DIRS now looks very different (example below)

```python
TEMPLATES = [  
{
    'BACKEND': 'django.template.backends.django.DjangoTemplates',
    'DIRS': [os.path.join(BASE_DIR, 'templates')],
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
```

- `MIDDLEWARE_CLASSES` is now just `MIDDLEWARE`

### Changes to urls.py

- Some internal packages have changed, patterns no longer exists or is not necessary in django.conf.urls.defaults. There is also a newer element called settings that will need to be
- You also now have to import your other apps directly if you're going to reference them in a single `urls.py`

```python
from django.conf import settings  
from django.conf.urls import url, include  
from django.contrib.staticfiles import views  
from django.conf.urls.static import static  
from django.contrib import admin  
from django.contrib.auth import views as auth_views  
from credoapp.api import BCSResource, ResponseResource  
import your_other_app  
from your_other_app import views
```

- urlpatterns no longer uses patterns('',...)

```python
# Before 
# urlpatterns = patterns('', ...,)`
# Now 
urlpatterns = [...,]  
```

- Quotations or ticks surrounding url view definitions are no longer necessary

```python
# Before
# url(r'^$', 'your_other_app.views.index', name='home')
# Now
url(r'^$', your_other_app.views.index, name='home')  
```

- The syntax for django.contrib.auth.views.login and .logout is slightly different

```python
# Before
# url(r'^login/$', 'django.contrib.auth.views.login', {'template_name': 'login.html'}),
# url(r'^logout/$', 'django.contrib.auth.views.logout', { 'next_page': '/login/'}),
# Now
url(r'^login/$', auth_views.login),  
url(r'^logout/$', auth_views.logout, { 'next_page': '/login/'})  
```

- `django.contrib.auth.views.login` also expects a template for your login page in your templates directory under `registration/login.html`

### Changes to Forms

Forms now require fields to be specified, in my case this was really straightforward, I just needed to add `fields = '__all__'` to my form classes.

```python
# Before
class SomeModelForm(forms.ModelForm)  
    class Meta:
        model = SomeModel   
# Now
class SomeModelForm(forms.ModelForm)  
    class Meta:
        model = SomeModel   
        fields = "__all__" 
```

### Changes to models.py

- `email_re` was removed, in favour of `validate_email`. No validation raise error required, `validate_email` does this automatically.

```python
# Before
# from django.core.validators import email_re
# Now 
from django.core.validators import validate_email  
...
# Before
# email_re.match(email) 
# Now
validate_email(email)
```

### Changes to Templates in General

- References to urls need to be changed using their namespace (if applicable, not in my case) or using their name.
- References to urls need to be changed using their name, and should be quoted
- These can exist everywhere, recommend using find in files % url to find them all and be methodical!

```HTML
# Before
# <a href="{% url someapp.views.view_name kw1 %}
# Now
<a href="{% url 'view_name' kw1 %} 
``` 
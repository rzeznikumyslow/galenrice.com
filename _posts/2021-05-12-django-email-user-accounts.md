---
title: Django user account login by email (no usernames)
author: Galen Rice
date: 2021-05-12 01:00:00 -0400
categories: [Code, Django]
tags: [django, python, code, site building]
---

A common gripe with the Django framework is how it handles user accounts. Or, rather, how difficult it is to change from its default `User` model to something else.

Django's default system uses username-based logins, but a lot of developers are looking to use email-based logins, which have become commonplace across the web. Not only does this force a user to provide that email on signup (Django's default auth system does not require the email field), it also simplifies the sign-up process itself (users may not be in the mood to pick a username for every site they visit).

So, if you're looking to use Django but have email-based logins, this post is for you.

Unfortunately, the process is...

## More complex than expected

Taking a look at [Django's docs on customizing user authentication][customizing_user_auth], one may quickly feel overwhelmed. Django's documentation is one of the best resources around and should be the first place one looks for answers, but it is certainly not a step-by-step tutorial (though their [official tutorial][django_tutorial] is still one of the best ways to get acquainted with the framework overall).

Could a tutorial like the one below be added to Django's docs? Honestly, *probably not*. Authentication is a difficult topic in any web technology, and there is no "one size fits all" solution. What I present below is an opinion ([one of many (DuckDuckGo search)][ddg_search_auth_email]).

I hope you'll find this useful, offer suggestions for improvements if you disagree, or use this as a future reference for your other projects (speaking to future-me here).

## Getting started

First up, we'll of course need a Django project running. If you have one already, great! Skip ahead to [Step 1](#step-1-create-an-app-to-contain-your-authentication-system).

### Step 0: Setting up your project

To start a fresh Django project, as with any new Python project, you should first create and activate a new virtual environment, preferably with the [latest version of Python][python.org] available.

There are many options for dependency management out there. [This SO answer][python_venv_so_answer] does a great job comparing them all. Feel free to spend time exploring them (my personal preference is [poetry], for what that's worth).

For simplicity, we'll use the built-in `venv` module:

- On Linux:

  ```shell
  # Make a new python venv:
  $ python3 -m venv myvenv
  # and activate it:
  $ source myvenv/bin/activate
  ```

- On Windows (using PowerShell):

  ```shell
  # Make a new python venv:
  $ py -m venv myvenv
  # and activate it:
  $ .\myvenv\Scripts\activate
  ```

Once the venv is activated, all commands are the same.

```shell
# Update pip:
(myvenv) $ python -m pip install --update pip
# Install latest Django:
(myvenv) $ pip install Django
# Use `django-admin` to start a new project:
(myvenv) $ django-admin startproject myproject
(myvenv) $ cd myproject
```

> Note that your prompt should start with `(myvenv)`, as shown above; indicating the virtual environment has been activated.

Normally at this stage in a new Django project, you would run some other management commands to get things started, such as `migrate` and `createsuperuser`, but **don't run those yet!** The changes we'll be making below need to have migrations run *before* the migrations of some of Django's built-in apps (see the note at the start of Step 1 for details).

### Step 1: Create an app to contain your authentication system

> **Note**: If you're making these changes in an existing project, see [here][change_model_mid_project] for details on how to set up a custom user model "mid-project". What we'll do below is best achieved *before* the migrations from other apps are executed.

> The remedy for changing your User model "mid-project" involves re-ordering migrations and/or manually fixing the database schema, neither of which are particularly fun. Try your best to do all this **first** in your new project.

Customizing the `User` model to use `email` for logins requires making a new model altogether, and that model must live inside some Django "app". You can, of course, add the model to any app your already have, but I find it best to contain the model and its logic somewhere else, away from your site's business logic.

What you name the new app is up to you. I prefer `accounts`:

```shell
# Remember you activate your venv, and
# cd into the project to find the `manage.py` module!
(myvenv) $ python manage.py startapp accounts
```

As with any new Django app, be sure to include it in `INSTALLED_APPS` in your `settings.py` module. Otherwise, Django won't know the new model exists.

```python
# mysite/settings.py

INSTALLED_APPS = [
    ...
    "accounts",
]
```

### Step 2: Create a new FooUser model

Now we'll create a model to use in place of Django's default `User`. We can call the new model `CustomUser` or `MyUser` or `SurprisinglyHandsomeUserPerson` or whatever else.

In my own projects, I just name it `User` for consistency. However, to prevent any confusion in this demonstration, we'll call it `FooUser`.

Open up `accounts/models.py` and add the following:

```python
# accounts/models.py

from django.contrib.auth.models import AbstractUser

class FooUser(AbstractUser):
    pass
```

We will build upon this as we go.

For now, return to `settings.py` and add the setting for `AUTH_USER_MODEL` to point to our new model:

```python
# mysite/settings.py

AUTH_USER_MODEL = "accounts.FooUser"
```

See [here][substitute_custom_user_model] for details on this setting.

#### A word on AbstractUser vs AbstractBaseUser

You'll see I'm subclassing the new `FooUser` from `AbstractUser`, which is a fairly common solution. The other option is to use `AbstractBaseUser`.

Why should we use one vs the other? A brief detour into Django's source code gives us a hint:

- [`AbstractBaseUser`][abstract_base_user_gh_link] provides the core machinery for authentication, namely password handling. Everything else, you have to provide yourself, including the username identifier, an email field, and so on. These *must* be implemented in order to have a working User model.
- [`AbstractUser`][abstract_user_gh_link] builds on `AbstractBaseUser` to add those required fields and more, including username, email, `is_staff`, `is_active`, etc. It also subclasses from [`PermissionsMixin`][permissions_mixin_gh_link], adding `is_superuser` and connections to the Groups and Permissions models that are a key part of Django's auth system.

The built-in `User` model is little more than a subclass of `AbstractUser`. So, if what we want is Django's `User` with tweaks, then `AbstractUser` is the way to go.

On the other hand, if you know precisely how you want authentication to be handled in your Django site, by all means start by subclassing `AbstractBaseUser` and start hacking away.

### Step 3: Adjust the FooUser model to our needs

Getting back on topic, we've got `FooUser`, but so far it's just a shell that mimics the original `User` model. Now, let's write some new logic to change things up.

1. First up, the `email` field on the new model needs to be unique for the model, to prevent duplicate users. We can do this by redefining the `email` field like so:

   ```python
   # accounts/models.py

   # Add these imports:
   from django.db import models
   from django.utils.translation import gettext_lazy as _

   class FooUser(AbstractUser):
       email = models.EmailField(
           _("Email address"),
           unique=True,
           error_messages={
               "unique": _("This email address is already in use.")
           },
       )
   ```

   Note the addition of `gettext_lazy`, which allows you to integrate [translation features][django_translation]. It's a good idea to include these calls early in the project, creating the hooks for translating content without massive refactors later.

   The key difference between our new `email` field and the old one is the addition of `unique=True`, ensuring no two users use the same email. The original version would also set `blank=True`, making the field optional in Django's Admin and other forms; but we omit this to force it to be required.

   As a bonus, we add a useful error message for when that uniqueness constraint is violated.

2. Next we need to tell Django to look at the `email` field as the username identifier for this model. We do this with [`USERNAME_FIELD`][username_field] and simply name the field we want as a string:

   ```python
   # accounts/models.py

   class FooUser(AbstractUser):
       ...
       USERNAME_FIELD = "email"
   ```

3. `AbstractUser` also sets `email` as one of its [`REQUIRED_FIELDS`][required_fields], used when prompting for user details during the `createsuperuser` command. This would appear to make sense - we do require the email field - but it actually generates an error if left alone:

   ```
   (myvenv) $ python manage.py superuser
   ...
   argparse.ArgumentError: argument --email: conflicting option string: --email
   ```

   The docs for `REQUIRED_FIELDS` actually state that it "should *not* contain the USERNAME_FIELD", and it appears this is why. Therefore, we must remove it, by setting `REQUIRED_FIELDS` to an empty list:

   ```python
   # accounts/models.py

   class FooUser(AbstractUser):
       ...
       REQUIRED_FIELDS = []
   ```

   If you want to further customize the User model and add new fields, you can add those fields to this list later to have them prompted for when running `createsuperuser`.

4. Next, `AbstractUser` provides the `username` field definition, but we don't want that anymore. Some folks will make this field optional and allow username entry in some capacity, but I prefer to remove it entirely.

   Doing that is as simple as setting `username` to `None` at the class level, so no field gets defined at all:

   ```python
   # accounts/models.py

   class FooUser(AbstractUser):
       ...
       username = None
   ```

## The completed FooUser model

So far so good! You should now have a full model that looks like so:

```python
# accounts/models.py

from django.contrib.auth.models import AbstractUser
from django.db import models
from django.utils.translation import gettext_lazy as _


class FooUser(AbstractUser):
    """Our custom user model."""

    username = None
    email = models.EmailField(
        _("Email address"),
        unique=True,
        error_messages={
            "unique": _("This email address is already in use.")
        },
    )

    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = []
```

## Customizing FooUser's model manager

Before we start migrating this model and creating superusers, though, we'll need to adjust the [model manager][django_model_managers] a bit.

If left as-is, the manager tied to `AbstractUser` will attempt to create users with the `username` field, which we no longer have. The same issue would occur in the Django Admin if we attempted to create the new user there, as well.

To remedy this, we'll write a new model manager for `FooUser`. This is less daunting than it seems, despite the complexity of the code you're about to see.

Simply create a new `managers.py` module in the `accounts/` directory, then add the following:

```python
# accounts/managers.py

from django.contrib.auth.models import UserManager
from django.contrib.auth.hashers import make_password


class FooUserManager(UserManager):
    """Overrides to allow authentication by email, not username."""

    use_in_migrations = True

    def create_user(self, email, password=None, **extra_fields):
        extra_fields.setdefault("is_staff", False)
        extra_fields.setdefault("is_superuser", False)

        if not email:
            raise ValueError("Email field is required")

        email = self.normalize_email(email)
        user = self.model(email=email, **extra_fields)
        user.password = make_password(password)
        user.save(using=self._db)
        return user

    def create_superuser(self, email, password, **extra_fields):
        extra_fields.setdefault("is_staff", True)
        extra_fields.setdefault("is_superuser", True)
        extra_fields.setdefault("is_active", True)
        return self.create_user(email, password, **extra_fields)
```

Connect this manager to the `FooUser` model by adding it as the `objects` manager, like so:

```python
# accounts/models.py

from .managers import FooUserManager

class FooUser(AbstractUser):
    ...
    objects = FooUserManager()
```

Let's break this down a bit (or you can [skip to the next section](#migrations-and-app-startup) if you just want to get on with things 😊 ).

Django's `UserManager` takes care of user creation in general. The `create_superuser()` method is called when you use the command `manage.py createsuperuser` to generate a superuser from the command line. The original method (as well as the other `create_user()` method) has a `username` positional argument; but since we no longer have a username, this would throw an error if left unchecked.

The above code is very similar to what Django uses for these methods ([see for yourself!][django_gh_usermanager]), so we can be confident we're not missing any steps. The other available method, `with_perm`, is left completely alone, so we can re-use that in our app if we choose.

## Migrations and app startup

With the above in place, we're ready to complete the startup steps for your new Django site.

1. First create the migration for the `FooUser` model, using the `makemigrations` command:

   ```shell
   # Remember to activate the venv first!
   (myvenv) $ python manage.py makemigrations
   Migrations for 'accounts':
     accounts/migrations/0001_initial.py
       - Create model User
   ```

2. Next, we can finally run all migrations to initialize the database for the first time, using the `migrate` command:

   ```shell
   (myvenv) $ python manage.py migrate
   Operations to perform:
     Apply all migrations: accounts, admin, auth, contenttypes, sessions
     Applying contenttypes.0001_initial... OK
     Applying contenttypes.0002_remove_content_type_name... OK
     Applying auth.0001_initial... OK
     Applying auth.0002_alter_permission_name_max_length... OK
     Applying auth.0003_alter_user_email_max_length... OK
     Applying auth.0004_alter_user_username_opts... OK
     Applying auth.0005_alter_user_last_login_null... OK
     Applying auth.0006_require_contenttypes_0002... OK
     Applying auth.0007_alter_validators_add_error_messages... OK
     Applying auth.0008_alter_user_username_max_length... OK
     Applying auth.0009_alter_user_last_name_max_length... OK
     Applying auth.0010_alter_group_name_max_length... OK
     Applying auth.0012_alter_user_first_name_max_length... OK
     Applying accounts.0001_initial... OK
     Applying admin.0001_initial... OK
     Applying admin.0002_logentry_remove_auto_add... OK
     Applying admin.0003_logentry_add_action_flag_choices... OK
     Applying sessions.0001_initial... OK
   ```

   You can see from this output that our `accounts.0001_initial` migration ran *before* migrations for `admin` and `sessions`, both of which are core Django apps. This is why setting up the User model early in the project lifecycle was crucial.

3. Once `migrate` completes, you can run the `createsuperuser` command to make your first superuser account:

   ```shell
   (myvenv) $ python manage.py createsuperuser
   Email address: myuser@example.com
   Password:
   Password (again):
   Superuser created successfully.
   ```

4. Finally, start up the development server using `runserver`:

   ```shell
   (myvenv) $ python manage.py runserver
   ```

You should now be able to visit `http://localhost:8000` to see the initial site, and `http://localhost:8000/admin` to login to Django Admin using your new superuser account.

## Updating the Admin site

To get our `FooUser` model working in Django's Admin site, we need to make a custom `ModelAdmin` class to register it.

The standard model uses `django.contrib.auth.admin.UserAdmin`, so we can just subclass this to take advantage of most of its machinery, adjusting only what we need.

For starters, let's simply create the class and register it with our `FooUser` model:

```python
# accounts/admin.py

from django.contrib import admin
from django.contrib.auth.admin import UserAdmin

from .models import FooUser


class FooUserAdmin(UserAdmin):
    pass


admin.site.register(FooUser, FooUserAdmin)
```

If the server is running, you will probably see an error in your console at this point:

```shell
ERRORS:
<class 'accounts.admin.FooUserAdmin'>: (admin.E033) The value of 'ordering[0]' refers to 'username', which is not an attribute of 'accounts.FooUser'.
```

So, there's more to be adjusted before we can complete this.

1. The old `username` field is referenced in `UserAdmin` under [`list_display`][django_admin_list_display], [`search_fields`][django_admin_search_fields], and [`ordering`][django_admin_ordering]. So, we copy over the contents of these settings, omitting `username` and focusing on our remaining fields:

   ```python
   # accounts/admin.py

   class FooUserAdmin(UserAdmin):
       ...
       list_display = ("email", "first_name", "last_name", "is_staff")
       search_fields = ("email", "first_name", "last_name")
       ordering = ("email",)  # Don't forget that comma, or it won't be a tuple!
   ```

   In the case of `ordering` in particular, `username` was its only member before, so we'll just add `email` there, instead.

2. [`fieldsets`][django_admin_fieldsets] needs to be adjusted. The first section needs to focus on our required `email` field instead of `username`, and a subsequent `Personal Info` section needs to have `email` removed from it. The rest we'll leave unchanged.

   Those changes look like so:

   ```python
   # accounts/admin.py

   # Add this import:
   from django.utils.translation import gettext_lazy as _

   class FooUserAdmin(UserAdmin):
       ...
       fieldsets = (
           (None, {
               "fields": ("email", "password"),
           }),
           (_("Personal info"), {
               "fields": ("first_name", "last_name"),
           }),
           *UserAdmin.fieldsets[2:],
       )
   ```

   Note that last line, `*UserAdmin.fieldsets[2:]`. Since `UserAdmin` has defined `fieldsets` already, we can slice that tuple to retrieve the parts we aren't changing (everything after the first two sections); then unpack that slice back into `FooUserAdmin.fieldsets` using the "splat" operator (`*`).

3. Finally, `UserAdmin` has a custom attribute similar to `fieldsets` called `add_fieldsets`. This is used in place of `fieldsets` when creating a new user (using some extra logic in the [`get_fieldsets`][django_admin_get_fieldsets] method). This is how the User Admin displays a simpler form with just `username` and two `password` fields, only providing all other entries when the User has already been created.

   We adjust this to use our `email` field in place of `username`, like so:

   ```python
   # accounts/admin.py

   class FooUserAdmin(UserAdmin):
       ...
       add_fieldsets = (
           (None, {
               "classes": ("wide",),
               "fields": ("email", "password1", "password2"),
           }),
       )
   ```

With that, we should have a fully-functional Admin page for `FooUser`. The full code looks like so:

```python
# accounts/admin.py

from django.contrib import admin
from django.contrib.auth.admin import UserAdmin
from django.utils.translation import gettext_lazy as _

from .models import FooUser

class FooUserAdmin(UserAdmin):
    """Custom admin class for our FooUser model."""

    list_display = ("email", "first_name", "last_name", "is_staff")
    search_fields = ("email", "first_name", "last_name")
    ordering = ("email",)

    fieldsets = (
        (None, {
            "fields": ("email", "password"),
        }),
        (_("Personal info"), {
            "fields": ("first_name", "last_name"),
        }),
        *UserAdmin.fieldsets[2:],
    )
    add_fieldsets = (
        (None, {
            "classes": ("wide",),
            "fields": ("email", "password1", "password2"),
        }),
    )


admin.site.register(FooUser, FooUserAdmin)
```

## Using your new model in other apps

A lot of times you'll find example code that refers to the *original* User model from `django.contrib.auth.models` by importing it directly:

```python
# DON'T DO THIS!
from django.contrib.auth.models import User
```

Doing so negates the benefits of your custom model. Even using this import in a project that otherwise does not change the User model simply makes it more difficult for that project to change to a custom User model later, since you'll need to locate and refactor every instance of this import as well as the code using it.

Django includes a standard way to access the current user model in any context, no matter which model - theirs, yours, [allauth's][django_allauth], whomever's - you choose to implement. This standard method is the `get_user_model` function:

```python
# This is the way:
from django.contrib.auth import get_user_model

User = get_user_model()
```

With our `FooUser` model implemented, `get_user_model()` will return the `FooUser` class, then assign that class to `User`. From there, you can use `User` like you would the standard Django User model: retrieving user instances, creating users, authenticating, deleting, and so on.

You can use this same reference as the target for a relational field, such as `models.ForeignKey`. However, it's typically better to use `settings.AUTH_USER_MODEL`, instead:

```python
from django.conf import settings
from django.db import models

class MyNewModel(models.Model):
    my_user = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
    )
```

This is because `ForeignKey` and other relational fields can accept a string value for the target model, in the same format as `settings.AUTH_USER_MODEL`. This method is [recommended by Django docs][django_auth_referencing_user_model], as well.

## Bonus: UUID primary keys for everyone!

My personal preference is to use UUIDs as the primary key in most of my models in new projects, including the User model. If nothing else, it settles squabbles about who gets to be "#1" in the new application (instead, you're `89b6504c-d0b6-4f34-adab-a4034413c5aa` and that's final /s).

Implementing UUIDs in Django models is pretty straightforward. Simply use [`UUIDField`][django_uuidfield] and provide Python's built-in `uuid.uuid4` function as a default:

```python
import uuid

from django.db import models


class MyModel(models.Model):
    uuid = models.UUIDField(
        default=uuid.uuid4,
        editable=False,
    )
```

You can take this a couple steps further by:

1. assigning the field to the `id` attribute of the model (usually set to an AutoField) with `primary_key` set to `True`; and
2. applying this logic to an abstract model that you can use as a mixin anywhere in your project.

My projects often use a `core` app that houses `settings.py` and `wsgi.py` and the like. This is where I would then place a new `models.py` module which *only* contains abstract models (since the "core" app is not a real Django app, it won't be able to use non-abstract or "concrete" models, anyway).

Thus, the abstract model `UUIDPrimaryKeyModel`:

```python
# core/models.py

import uuid

from django.db import models


class UUIDPrimaryKeyModel(models.Model):
    """Defines `id` as a UUIDField for the primary key."""

    id = models.UUIDField(
        primary_key=True,
        default=uuid.uuid4,
        editable=False,
    )

    class Meta:
        abstract = True
```

You can then very easily add this to the `FooUser` model from earlier, simply by adding it as a subclass:

```python
# accounts/models.py

from core.models import UUIDPrimaryKeyModel

...

class FooUser(UUIDPrimaryKeyModel, AbstractUser):
    ...
```

And presto, your custom User model is keyed by UUID values automatically, with no additional customization needed.

## Wrapping up

In this article, we:

- Created a custom User model, `FooUser`, based on Django's `AbstractUser`; removing the `username` field and adjusting the model to use `email` as the new username.
- Created a custom model manager, `FooUserManager`, that correctly creates new users and superusers without referencing the old `username` field; and connected this manager to `FooUser`.
- Registered `FooUser` with the Admin site by re-using much of the original `UserAdmin`, removing references to `username` and ensuring `email` was brought to the top.
- Briefly went over how to properly use the new `FooUser` in other parts of a Django project.
- As a bonus, set up `FooUser` to use a UUID primary key field by way of an abstract model.

I hope you enjoyed reading this. Feel free to hit me up using the contact links in the side bar if you have questions or corrections to share!

[abstract_base_user_gh_link]: https://github.com/django/django/blob/7582d913e7db7f32e4cdcfafc177aa77cbbf4332/django/contrib/auth/base_user.py#L47
[abstract_user_gh_link]: https://github.com/django/django/blob/7582d913e7db7f32e4cdcfafc177aa77cbbf4332/django/contrib/auth/models.py#L321
[change_model_mid_project]: https://docs.djangoproject.com/en/3.2/topics/auth/customizing/#changing-to-a-custom-user-model-mid-project
[customizing_user_auth]: https://docs.djangoproject.com/en/3.2/topics/auth/customizing/
[ddg_search_auth_email]: https://duckduckgo.com/?q=django+user+auth+email
[django_admin_fieldsets]: https://docs.djangoproject.com/en/3.2/ref/contrib/admin/#django.contrib.admin.ModelAdmin.fieldsets
[django_admin_get_fieldsets]: https://docs.djangoproject.com/en/3.2/ref/contrib/admin/#django.contrib.admin.ModelAdmin.get_fieldsets
[django_admin_list_display]: https://docs.djangoproject.com/en/3.2/ref/contrib/admin/#django.contrib.admin.ModelAdmin.list_display
[django_admin_ordering]: https://docs.djangoproject.com/en/3.2/ref/contrib/admin/#django.contrib.admin.ModelAdmin.ordering
[django_admin_search_fields]: https://docs.djangoproject.com/en/3.2/ref/contrib/admin/#django.contrib.admin.ModelAdmin.search_fields
[django_allauth]: https://django-allauth.readthedocs.io/en/latest/
[django_auth_referencing_user_model]: https://docs.djangoproject.com/en/3.2/topics/auth/customizing/#referencing-the-user-model
[django_gh_usermanager]: https://github.com/django/django/blob/7582d913e7db7f32e4cdcfafc177aa77cbbf4332/django/contrib/auth/models.py#L129-L189
[django_model_managers]: https://docs.djangoproject.com/en/3.2/topics/db/managers/
[django_translation]: https://docs.djangoproject.com/en/3.2/topics/i18n/translation/
[django_tutorial]: https://docs.djangoproject.com/en/3.2/intro/tutorial01/
[django_uuidfield]: https://docs.djangoproject.com/en/3.2/ref/models/fields/#uuidfield
[permissions_mixin_gh_link]: https://github.com/django/django/blob/7582d913e7db7f32e4cdcfafc177aa77cbbf4332/django/contrib/auth/models.py#L232
[poetry]: https://python-poetry.org/
[python_venv_so_answer]: https://stackoverflow.com/a/41573588
[python.org]: https://www.python.org/
[required_fields]: https://docs.djangoproject.com/en/3.2/topics/auth/customizing/#django.contrib.auth.models.CustomUser.REQUIRED_FIELDS
[substitute_custom_user_model]: https://docs.djangoproject.com/en/3.2/topics/auth/customizing/#substituting-a-custom-user-model
[username_field]: https://docs.djangoproject.com/en/3.2/topics/auth/customizing/#django.contrib.auth.models.CustomUser.USERNAME_FIELD

---
title: Node development for the Django dev
author: Galen Rice
date: 2022-01-20 13:00:00 -0500
tags: [django, python, node, javascript, tailwindcss, code, site building]
---

Django is a great framework for creating a backend and a mostly-static frontend using its template system. Projects like [HTMX] are making it possible to stay mostly within the realm of that template system while adding greater functionality and interactivity, but still your site is mostly a backend.

Getting the most out of a site on the modern web requires some form of **JavaScript** interactivity. For devs who feel most comfortable in the Python ecosystem (like yours truly), making the jump into frontend development can be a scary proposition.

In this post, I'll offer some basic knowledge and techniques for getting started in frontend development from the perspective of a (primarily) Django backend developer. By no means am I an expert, and I encourage you to learn more about JavaScript and Node on your own; but you can use these strategies as a starting point.

This post also serves as a tutorial for creating a barebones Django site that includes a Node/NPM toolchain, specifically one that can build [TailwindCSS] and use it within Django templates.

**TL;DR**: Check out the [tutorial repo][tutorial-repo] for the final version of the project built for this post.

## Getting started: install Node.js

First thing's first, you will need [Node.js] on your machine. Head over there to download and install the latest version.

This part should be fairly straightforward for most users, so there's not much else to say here.

### Tip: On Windows, consider Chocolatey

If you are developing on a Windows machine, I highly recommend using [Chocolatey] for this part. This is a package manager for Windows programs that you can setup on any machine you have Admin rights to.

From a PowerShell terminal (opened as Administrator), you can install the latest version of Node.js using Chocolatey's `choco` utility like so:

```powershell
choco install nodejs
```

You can also install any other [Chocolatey package][chocolatey-packages] that you like. Personally, this is how I manage my [Python][chocolatey-python] and [Docker][chocolatey-docker] upgrades on my Windows machine.

Upgrading those packages is as simple as:

```powershell
choco upgrade all

# To skip confirmations and upgrade automatically, use the `-y` flag:
choco upgrade all -y

# To do nothing but show which packages need updates, use `--noop`:
choco upgrade all --noop
```

Using this, you main developer tools (including Node) will stay up-to-date without the fuss.

## Get familiar with NPM

[NPM], or **Node Package Manager**, is similar in function to Python's [pip]. I would argue that NPM is a bit more mature than pip in some regards, though it's not without its own issues.

Now, unless you're going to use Node as a backend service, most if not all commands you use in your development will go through NPM in some way. If you have never used NPM before, I highly recommend you pause reading here and check out this [Absolute Beginner's Guide to Using npm][npm-beginner-guide]. That guide (which is not mine), can walk you through:

- What the `package.json` file is and what it's used for.
- The basic metadata fields of `package.json`.
- The difference between `dependencies` and `devDependencies`.
- Running basic commands `npm init` to initialize a project and `npm install` to install packages.

Once you're done there, we'll build off that knowledge for our needs here.

## Adding NPM assets to a Django project

Now we'll get to the heart of the matter, how to mix the NPM peanut butter with our Django chocolate, so to speak. What follows is a tutorial in which we will:

- Create a basic Django project from scratch.
- Add NPM assets to the project in a separate directory.
- Install Tailwind.
- Configure Tailwind to use Django templates for content and output styles to Django's static files.
- Set up a basic template that uses the new styles.

### Step 0: Create a basic Django project

We'll begin with a new Django project generated with the `django-admin startproject` command from Django 4.0. I will assume you know how to set up a virtual environment, activate said environment, and `pip install Django` in order to get started.

In all, we'll run the following commands (inside our virtual environment) in the terminal:

```shell
# Generate the new project
django-admin startproject django_node_example

# cd into the top level directory of the project we just made
cd django_node_example

# migrate a new database (defaults to a SQLite database file)
python manage.py migrate

# run the development server on port 8000 (the default)
python manage.py runserver 8000
```

You can now open your browser to `http://localhost:8000`, and you should see the following (the "successful install" page from Django):

![Django initial site](/assets/img/2022/01/20/django-initial-site.png)

Quick and easy. You can shut down the development server (hit `Ctrl-C`) for now: we won't be needing it for a bit.

### Step 1: Adding our toolchain

To begin adding our Node assets, we first have to understand the basic project structure.

When we open the top-level directory for our project in VS Code, for example, the file structure should look like so:

![Django basic file structure](/assets/img/2022/01/20/django-startproject-files.png)

_Note: I am using [Rainglow] "Allure Contrast" color theme, as well as [Material Theme Icons][material-icons]._

Now we're going to add a new directory at the _top level_ of the project (the same as the _outermost_ `django_node_example/` directory), where all our Node assets will be contained. I like to call this new directory `js_toolchain` (which is the name I'll use moving forward in this post), but you can name it whatever makes sense for you.

```shell
# Return to the root directory
# (where you ran `django-admin startproject` command earlier)
cd ..

# Create our "toolchain" directory, then `cd` into it.
mkdir js_toolchain
cd js_toolchain
```

From here, we run `npm init` to generate the `package.json` file. This will prompt you for some starter values to fill in, as you'll see below. You can leave everything at their defaults, or enter details as you go along: you can always change them manually in `package.json` later.

```
$ npm init
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sensible defaults.

See `npm help init` for definitive documentation on these fields
and exactly what they do.

Use `npm install <pkg>` afterwards to install a package and
save it as a dependency in the package.json file.

Press ^C at any time to quit.
package name: (js_toolchain) django_node_example
version: (1.0.0) 0.0.1
description: Example of Node development within a Django project
entry point: (index.js)
test command:
git repository:
keywords:
author: Galen Rice
license: (ISC) MIT
About to write to /path/to/my/project/js_toolchain/package.json:

{
  "name": "django_node_example",
  "version": "0.0.1",
  "description": "Example of Node development within a Django project",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Galen Rice",
  "license": "MIT"
}


Is this OK? (yes)
$
```

The file generated looks identical to the example shown in the output (see above), and your project files should now look like this:

![Django files with js_toolchain added](/assets/img/2022/01/20/django-files-with-jstoolchain.png)

> **Note:** the `main` key and `test` script are not necessary for our purposes, and can be safely removed.

### Step 2: Adding a package - TailwindCSS

[TailwindCSS] is a popular CSS framework that focuses on utility classes that allow building components from scratch within HTML, rather than writing raw CSS or using pre-built components. Some love it, some hate it, but the fact is that setting it up can be a little painful for first-time users, especially Django developers who don't know NPM so well (i.e. the readers of this post, I assume 😊).

Let's begin by following the [installation instructions][tailwind-installation]. We'll focus on the Tailwind CLI method to reduce complexity for this example.

Navigate to your `js_toolchain` directory (containing `package.json`) and run the following:

```shell
# Install tailwindcss as a devDependency
npm install -D tailwindcss

# initialize tailwind
npx tailwindcss init
```

_Note the use of `npx`, not `npm`, for the second command. See details on the `npx` command [here][npx-command]._

You'll notice some new files in your project, including a `node_modules/` directory (which you should **not** commit to a Git repository, by the way!), a `package-lock.json` file, and `tailwind.config.js`.

Further, `package.json` has been updated with a `devDependencies` key containing the following:

```json
{
  "devDependencies": {
    "tailwindcss": "^3.0.15"
  }
}
```

_Your exact version may differ: Tailwind sees frequent patch updates._

Continuing with the Tailwind instructions, we'll edit our new `tailwind.config.js` file to update the `content` settings. There, we'll give it at least one [glob pattern] to tell it where to find the Django site's template files:

```javascript
/*  js_toolchain/tailwind.config.js  */
module.exports = {
  content: ["../django_node_example/**/*.html"],
  theme: {
    extend: {}
  },
  plugins: []
};
```

The basic pattern is to traverse _up_ to the root directory of the project (using `../`), _down_ to the Django site root (the directory containing `manage.py`), check every sub-directory (`**/`), and match every HTML file it finds (`*.html`).

Next, we'll create a new file, `js_toolchain/src/input.css`, which serves as the basis for our generated CSS. To that new file, we'll add the following:

```css
/*  js_toolchain/src/input.css  */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Finally, we can run the Tailwind CLI build process to test its output (remove the `--watch` option for the time being):

```shell
npx tailwindcss -i ./src/input.css -o ./dist/output.css
```

This will generate an `output.css` containing the _real_ CSS to be used in the project, within a new `js_toolchain/dist/` directory. If you successfully generated this file, you can safely delete it and the `dist/` directory, as we won't be using them moving forward.

At this stage, our project files look like this (after removing the `dist/` directory as mentioned):

![Django files with TailwindCSS set up](/assets/img/2022/01/20/django-tailwind-setup.png)

### Step 3: Adding a test page

We have TailwindCSS added, but it's not doing much for us when there are no templates in our example site. Time to change that!

Typically we would add templates to specific Django apps, but that's not strictly necessary. I like to add a `templates/` directory at the root of the project, mainly for the base template on which all pages rely or to have sub-templates with different site components (nav bars, headers, footers, etc.).

In this demo, we'll follow that same pattern by:

- Adding a single `index.html` template to the root of the Django site; and
- Setting up a single URL pointing to that template using a simple `TemplateView`.

At the root of the Django site (within the same directory that contains `manage.py`), add a new directory called `templates/`, then add an `index.html` file there with the following content:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <h1>Hello world!</h1>
  </body>
</html>
```

The project files should look like so (with `js_toolchain/` collapsed for now):

![Django files with root index.html template](/assets/img/2022/01/20/django-index-template.png)

Now, Django doesn't know how to find this `templates/` directory automatically. To do that, we need to adjust some of our settings. In the project's `settings.py` module, find the `TEMPLATES` variable, then add `BASE_DIR / 'templates'` to the `'DIRS'` list:

```python
# django_node_example/settings.py
TEMPLATES = {
    ...
    'DIRS': [
        BASE_DIR / 'templates',  # add this!
    ],
    ...
}
```

You'll notice `BASE_DIR` is a `pathlib.Path` object pointing to the root directory of the Django project (the one containing `manage.py`), so you can easily build paths off `BASE_DIR` to find the directories and other files you need.

Now, we'll edit the `urls.py` module found in the same directory as `settings.py` to add our route to the template. Here we'll use `django.views.generic.TemplateView` to create a simple route to our `index.html` template:

```python
# django_node_example/url.py
from django.contrib import admin
from django.urls import path
from django.views.generic import TemplateView  # add this!

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', TemplateView.as_view(template_name='index.html')),  # add this!
]
```

Now we simply start the server with `python manage.py runserver` and visit `http://localhost:8000` in our browser, and we should see our simple "Hello world!" output:

![Hello world output](/assets/img/2022/01/20/hello-world.png)

Our example doesn't look like much at the moment, mainly because it has absolutely zero styling (though some might argue [that's perfectly fine][mofo-website]). Let's get Tailwind involved and make things a bit neater.

### Step 4: Getting Tailwind static files in the right place

We need to put some building blocks together to make things work the way we want them to:

1. We need our `npx tailwindcss` command from earlier to be able to generate the proper styles.
2. The command needs to output our desired styles to a specific location where Django will pick them up as one of its [static files][django-static-files].
3. Django needs to be configured to pick up those centralized static files, similar to how we used a centralized `templates/` directory earlier.
4. We need to add the generated CSS in our `index.html` template using a `{% raw %}{% static %}{% endraw %}` template tag.
5. Finally, we need some Tailwind classes in `index.html` to demonstrate the process is working.

#### NPM package.json scripting 101

The first two concerns can be handled using the `'scripts'` key in our `js_toolchain/package.json` file. Here you can define any number of commands customized to your Node project needs. For our purposes, we'll add a `build` script like so:

```json
{
  "scripts": {
    "build": "npx tailwindcss -i ./src/input.css -o ../django_node_example/static/css/styles.css"
  }
}
```

This is similar to the example from the Tailwind installation, but this time we've changed the `-o` value so that it drops a `styles.css` file into `django_node_example/static/css/`, a directory within the Django site itself:

![Django files with styles.css generated](/assets/img/2022/01/20/django-styles-css-output.png)

You have total control over this scripting, including exactly what it does, what it's called ("build" or "build:dev" or "make_styles" or whatever), whether to chain multiple commands using `&&`, even whether to use one command to call another (i.e. `"npm run build && echo \"Built it!\""`).

You might recall the Tailwind install instructions had us using a `--watch` option in the build command? We can add that back now in a second command which we'll call `start`:

```json
{
  "scripts": {
    "build": "npx tailwindcss -i ./src/input.css -o ../django_node_example/static/css/styles.css",
    "start": "npx tailwindcss -i ./src/input.css -o ../django_node_example/static/css/styles.css --watch"
  }
}
```

This way, we have easy access to both a one-off process (`build`) and a server-like process that will aid us during development (`start`). Further down the line you might customize this further, with a build command intended for production use or one that includes minification. The possibilities are really endless here.

#### Django centralized static files

On to concern #3. We are outputting `styles.css` into a central `static/` directory at the project root, but Django (again) doesn't know how to find this automatically: its default assumes all your static files live within apps.

Open `settings.py` again. We need to add the `STATICFILES_DIRS` setting, which is omitted from the default project. This contains a list of paths that Django may search to locate static files, so we'll tell it how to find our new `static/` directory at the root of the Django project (defined by `BASE_DIR`).

```python
STATICFILES_DIRS = [
    BASE_DIR / 'static',
]
```

There's no need to add the `css/` sub-directory we generated: everything within that base `static/` directory will be included automatically. When we make use of these static files, we build off the "root" of these directories, so `django_node_example/static/css/styles.css` can be found in templates simply using `{% raw %}{% static 'css/styles.css' %}{% endraw %}`.

#### Updating our template

For the final two concerns (#4 and #5 above), we'll make edits to our `index.html` template.

First, we need to include our `styles.css` file in the `head` section, using a `{% raw %}{% static %}{% endraw %}` tag pointing to `css/styles.css`. And, in order to use `{% raw %}{% static %}{% endraw %}` tags, we have to include `{% raw %}{% load static %}{% endraw %}` somewhere near the top of the template.

```html
{% raw %}{% load static %}{% endraw %}
<!-- add this! -->

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />

    <!-- add this! -->
    <link
      rel="stylesheet"
      href="{% raw %}{% static 'css/styles.css' %}{% endraw %}"
    />

    <title>Document</title>
  </head>
</html>
```

Then, we do what we'll likely be doing for the rest of our frontend development within this application: using Tailwind classes to style our pages!

Start by running two concurrent terminals with the following commands:

```shell
# Terminal 1
cd django_node_example
python manage.py runserver 8000

# Terminal 2
cd django_node_example/js_toolchain
npm run start
```

This will start the development server for Django as well as our Tailwind build script, which will watch for changes to our templates and automatically rebuild `styles.css` for us.

Finally, edit `index.html` and change our simple `<h1>` tag to:

```html
<h1
  class="container mx-auto mt-8 p-2 border rounded text-3xl font-bold bg-purple-800 text-white text-center"
>
  Hello world!
</h1>
```

Save the file, the reload the browser window (at `http://localhost:8000`) and you should see this:

![Hello world styled with Tailwind](/assets/img/2022/01/20/hello-world-tailwind.png)

I am by no means a good UI designer, but the point stands that our Django project is now up and running with TailwindCSS styles! 🎉

## Expanding your toolchain

We've demonstrated a complete process that lets us pull NPM packages and get them playing nicely with our Django project, but the sky's the limit from here in terms of making better use of the NPM ecosystem. Some things you may consider adding:

### PostCSS

[PostCSS] gives you more flexibility in the CSS code you write, or for that matter the CSS that Tailwind or other packages outputs. Tailwind offers [instructions using PostCSS][tailwind-using-postcss], though it falls down slightly at the "use whatever command is configured" step of the process.

To augment our example project with PostCSS:

1. Install `postcss`, `postcss-cli`, and `autoprefixer` using NPM, the same way as you did for `tailwindcss`:

   ```shell
   npm install -D postcss postcss-cli autoprefixer
   ```

   `postcss-cli` is optional, but allows us to use `postcss` as a command in our build scripts later.

2. Create a `postcss.config.js` file in the `js_toolchain/` directory, with the following content:

   ```javascript
   module.exports = {
     plugins: {
       tailwindcss: {},
       autoprefixer: {}
     }
   };
   ```

3. In `package.json`, update the scripts to use `postcss` instead of `npx tailwindcss`. You may also want to allow it to pull a more generic set of source CSS, using `./src/**/*.css` as the first argument:

   ```json
   {
     "scripts": {
       "build": "postcss ./src/**/*.css -o ../django_node_example/static/css/styles.css",
       "start": "postcss ./src/**/*.css -o ../django_node_example/static/css/styles.css --watch"
     }
   }
   ```

That's it! You can start taking advantage of PostCSS and its vast ecosystem of tools to output cleaner cross-browser CSS, all from the same Tailwind source files.

### HTMX

As mentioned way at the top of this post, HTMX is another project you can take advantage of using your new NPM toolchain.

My preference is to install HTMX files from NPM, then use a [postinstall script][npm-postinstall] and the package [`copy-files-from-to`][copy-files-from-to] to move the necessary files from `node_modules/` out to the Django static files.

1. Install `htmx.org` and `copy-files-from-to` packages:

   ```shell
   npm install -D htmx.org copy-files-from-to
   ```

2. Add a `copyFiles` key to your `package.json` file containing the following:

   ```json
   {
     "copyFiles": [
       {
         "from": "node_modules/htmx.org/dist/**/*",
         "to": "../django_node_example/static/js/htmx.org/"
       }
     ]
   }
   ```

3. Add a `postinstall` script to the `scripts` section which simply calls `copy-files-from-to`:

   ```json
   {
     "scripts": {
       "postinstall": "copy-files-from-to"
     }
   }
   ```

   Now whenever you run an `npm install` or `npm ci` command on the project, the postinstall will automatically copy the static files for HTMX into the Django project.

4. Add the HTMX static files to your templates:

   ```html
   <head>
     <script src="{% raw %}{% static 'js/htmx.org/htmx.min.js' %}{% endraw %}"></script>
   </head>
   ```

Now you can use HTMX throughout your project.

#### Why not use the CDN version?

You can, of course, follow the Quick Start instructions on [htmx.org][htmx] and simply include `https://unpkg.com/htmx.org@1.6.1` (or latest version) in your site, thereby avoiding all the fuss. However, some projects prefer to vendor all their code to keep it local to the project, cutting down on I/O to external sources. Still others have hard requirements against using external sources, such as internal sites behind proxies that may block CDN sources entirely.

Personally, in at least one instance, my organization's team found that our team members in India were not able to load certain resources due to those proxies, while other teammates in Europe and the US still could. This led us to vendor all packages as much as possible, so that we could source them from our internal platform, on which there are no such restrictions (of course, after passing the packages through code scans and security audits, and pinning versions accordingly).

A final reason we'll get into in a moment involves updating packages automatically using services like [Dependabot]. These services (probably) will never find CDN links to packages in your code base. So, if you want to keep a package like HTMX updated, it is better to tag it as a devDependency in `package.json` and let Dependabot check for updates for you.

## Keeping things updated: Dependabot example

If you host your new project on GitHub, [Dependabot] is a handy service that can periodically check for dependency updates, then automatically create Pull Requests that update those dependencies on your behalf. With additional Continuous Integration workflows in place to perform appropriate scans and testing, you can ensure your project stays updated constantly, with little manual work required.

Further, you can use a single Dependabot configuration for all types of dependencies in your project, including the Python dependencies of your Django site _and_ the NPM dependencies of your new toolchain.

To get started, create a `.github/` directory (if you don't have one already) at the root of your repository (at the same level as the `django_node_example/` and `js_toolchain/` directories). In that new directory, create a `dependabot.yml` file containing:

```yaml
# To get started with Dependabot version updates, you'll need to specify which
# package ecosystems to update and where the package manifests are located.
# Please see the documentation for all configuration options:
# https://help.github.com/github/administering-a-repository/configuration-options-for-dependency-updates

version: 2
updates:
  # Python dependencies
  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "daily"
      time: "08:00"
      timezone: "US/Eastern"

  # Node dependencies
  - package-ecosystem: "npm"
    directory: "/js_toolchain"
    schedule:
      interval: "daily"
      time: "08:00"
      timezone: "US/Eastern"
```

_I recommend leaving the comments in place for your own reference._

This will check for Python dependency updates at the root of the project, so it will try to find `requirements.txt` or files from Pipenv, Poetry, or pip-compile. It also checks for your NPM dependencies located in `js_toolchain/`, so it will automatically pick up `package.json` and `package-lock.json` (it does also work with [Yarn], if that's your preference).

You can adjust the `schedule` settings as you see fit (see Dependabot's documentation, linked in the comments of the file contents above, for more details). The above works for me, so I get new PRs in the morning before I start working, but also at a time when my phone is active so I'll see the notifications coming in. 😊

Simply save and commit this to the main branch in your GitHub repository, and Dependabot will start checking for dependency updates automatically according to your `schedule` settings.

> **Tip**: Dependabot can also check for updates in GitHub Actions workflows and Dockerfiles, using `package-ecosystem: "github-actions"` and `package-ecosystem: "docker"` respectively. I recommend setting these up, as well, just to be safe.

## Putting it all together

You can check out my [tutorial repo][tutorial-repo] for the final version of the Django project built for this post. The only additions not discussed here include:

- Basic project files, such as `LICENSE`, `README.md`, and `.gitignore`.
- `pyproject.toml` and `poetry.lock`, used for installing dependencies with [Poetry].

Installation and startup instructions are included in the README of the tutorial repo itself.

Thanks for reading! 👋

[chocolatey-docker]: https://community.chocolatey.org/packages/docker-desktop
[chocolatey-packages]: https://community.chocolatey.org/packages/
[chocolatey-python]: https://community.chocolatey.org/packages/python/
[chocolatey]: https://chocolatey.org/
[copy-files-from-to]: https://www.npmjs.com/package/copy-files-from-to
[dependabot]: https://docs.github.com/en/code-security/supply-chain-security/keeping-your-dependencies-updated-automatically/about-dependabot-version-updates
[django-static-files]: https://docs.djangoproject.com/en/4.0/howto/static-files/
[django-tailwind]: https://github.com/timonweb/django-tailwind
[glob pattern]: https://en.wikipedia.org/wiki/Glob_%28programming%29
[htmx]: https://htmx.org/
[material-icons]: https://marketplace.visualstudio.com/items?itemName=Equinusocio.vsc-material-theme-icons
[mofo-website]: https://motherfuckingwebsite.com/
[node.js]: https://nodejs.org/en/
[npm-beginner-guide]: https://nodesource.com/blog/an-absolute-beginners-guide-to-using-npm/
[npm-postinstall]: https://docs.npmjs.com/cli/v8/using-npm/scripts#pre--post-scripts
[npm]: https://www.npmjs.com/
[npx-command]: https://docs.npmjs.com/cli/v8/commands/npx
[pip]: https://pip.pypa.io/en/stable/
[poetry]: https://github.com/python-poetry/poetry
[postcss]: https://postcss.org/
[rainglow]: https://marketplace.visualstudio.com/items?itemName=daylerees.rainglow
[tailwind-installation]: https://tailwindcss.com/docs/installation
[tailwind-using-postcss]: https://tailwindcss.com/docs/installation/using-postcss
[tailwindcss]: https://tailwindcss.com/
[tutorial-repo]: https://github.com/GriceTurrble/django-node-tutorial-final
[yarn]: https://yarnpkg.com/

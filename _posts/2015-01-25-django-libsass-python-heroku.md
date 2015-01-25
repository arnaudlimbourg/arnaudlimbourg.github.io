---
title: "Django application with libsass-python, minification and compression on heroku"
---

Now that the SEO title is out of the way, say you want to deploy a django app to Heroku.
You also use SASS files and would like 
to use [django-pipeline](https://github.com/cyberdelia/django-pipeline/) to manage the assets. You also want to use cssmin and uglifyjs 
to minify and compress your css and javascript files.

You would also prefer to not have to build on your machine and commit 
the files to your repository. You can always ```git reset``` but where is the beauty?

There are unfortunately not that many ressources on this subject. This post is an attempt 
to remedy that and will show one way it can be achieved.

So the goal is to have a django app deployed to heroku with SASS compiling using libsass-python, 
using [django-pipeline](https://github.com/cyberdelia/django-pipeline/), running 
cssmin and uglifyjs for production, all of that happening on the collectstatic step.

We will also host assets directly on the dyno thanks to [whitenoise](http://whitenoise.evans.io/en/latest/), 
and use cloudfront (or another CDN).

To achieve all of the above you will want to use the multi-buildpack feature. 
First we need the nodejs buildpack to have cssmin and uglifyjs and second we need 
the python buildpack for django and libsass-python.

## Part 1 : Django
Create an empty directory and create a django app. You can use [cookicutter](https://github.com/pydanny/cookiecutter-django.git) 
for example.

```bash
cookiecutter https://github.com/pydanny/cookiecutter-django.git
# fill in the fields
```

You now have your sample project. You can add the necessary requirements for pip.

```text
# add those to requirements/base.txt
whitenoise==1.0.6
libsass==0.6.2
django-pipeline==1.4.3
```

### SASS
Configure pipeline by editing relevant files : #c0d9289. This is 
where you put the libsass configuration.

We use the sassc command-line tool provided by the project. This is fairly straightforward.

## Part 2 : Heroku
Now we can move on to the heroku part proper. Create an app if you don't have one.

Configure buildpack.

```bash
heroku config:set BUILDPACK_URL=https://github.com/heroku/heroku-buildpack-multi.git
```

You then add a .buildpacks file to indicate which buildpack to use. 
We will use the nodejs one and a fork of the python one that forces pythonpath for collectstatic. 
When using the cookiecutter layout it is necessary to have the collectstatic command work.

You need to configure your STATIC_URL for production. 
If using [django-configurations](http://django-configurations.readthedocs.org/en/latest/) 
set it to a SecretValue, that way it has to be se as an environment variable.

```python
STATIC_URL = values.SecretValue()
```

for example:

```bash
heroku config:set DJANGO_STATIC_URL=https://d11111.cloudfront.net/static/
```

Set other needed variables if you used the cookiecutter example project otherwise ignore.

```bash
heroku config:set DJANGO_SECRET_KEY=insert_key_here
heroku addons:add heroku-postgresql:dev
heroku pg:promote DATABASE_URL
heroku config:set DJANGO_CONFIGURATION=Production
heroku config:set DJANGO_AWS_STORAGE_BUCKET_NAME=example
heroku config:set DJANGO_AWS_ACCESS_KEY_ID=example
heroku config:set DJANGO_AWS_SECRET_ACCESS_KEY=example
heroku config:set DJANGO_EMAIL_HOST_USER=example
```

To have whitenoise and pipeline work together you need to setup a custom storage that mixes all of them. 
For example you can use this django 1.7+ storage.

````python
from django.contrib.staticfiles.storage import ManifestStaticFilesStorage

from whitenoise.django import GzipStaticFilesMixin

from pipeline.storage import PipelineMixin


class WhiteNoisePipeline(GzipStaticFilesMixin, PipelineMixin, ManifestStaticFilesStorage):
    pass
```

Change your settings to use this storage.

```python
STATICFILES_STORAGE = 'heroku-libsass-python.storage.WhiteNoisePipeline'
```

## Compression and minification

This is one of the most annoying bit to get working. This is needed because paths variables are not set yet 
in the build stage as far as I understood.

In order to get it working you need to use the following settings for production. In the cookie-cutter 
example it means editing the ```production.py```.

```python
    PIPELINE_UGLIFYJS_BINARY = 'PATH=/app/.heroku/node/bin:$PATH /app/node_modules/.bin/uglifyjs'

    PIPELINE_CSSMIN_BINARY = 'PATH=/app/.heroku/node/bin:$PATH /app/node_modules/.bin/cssmin'
```

## Conclusion
Push to heroku and see the magic happen.

Your assets will be compiled, compressed, gzipped and have far-future expire headers. 
This will make your deployment workflow simpler. I am aware you can use a shell script 
or other means to achieve this I like the simplicity of this, even if the setup takes a bit 
of time.

You can find a sample git project : https://github.com/arnaudlimbourg/heroku-libsass-python

I hope this will be useful.

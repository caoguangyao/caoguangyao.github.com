---
layout: post
title: "Django in sae"
date: 2012-10-18 13:30
comments: true
categories: 
---
###tools need installed first by order: ###

* python 2.6 or python 2.7
* django 1.2.7 or above
* mysql
* phpmyadmin
* apache2
* install-tools
* svn
* git
* sae python-sdk tools

start a django project:

	django-admin.py startproject project

start a app:
	
	django-admin.py startapp app
	
modify settings.py file the database config struct

	from os import environ
	import sae.const
	
	sae_environ = environ.get('APP_NAME','')
	
	MYSQL_DB = 'your_database_name' 
	MYSQL_USER = 'your_username' 
	MYSQL_PASS = 'your_password' 
	MYSQL_HOST_M = 'your_mysql_host' 
	MYSQL_HOST_S = 'your_mysql_host' 
	MYSQL_PORT = 'your_mysql_port' 

	if sae_environ:
    	MYSQL_DB = sae.const.MYSQL_DB 
    	MYSQL_USER = sae.const.MYSQL_USER 
    	MYSQL_PASS = sae.const.MYSQL_PASS 
    	MYSQL_HOST_M = sae.const.MYSQL_HOST 
    	MYSQL_HOST_S = sae.const.MYSQL_HOST_S 
    	MYSQL_PORT = sae.const.MYSQL_PORT

	DATABASES = { 
    	'default': { 
        'ENGINE': 'django.db.backends.mysql', 
        'NAME': MYSQL_DB, 
        'USER': MYSQL_USER, 
        'PASSWORD': MYSQL_PASS, 
        'HOST': MYSQL_HOST_M, 
        'PORT': MYSQL_PORT, 
    	} 
	}

don't foget to add app to INSTALLED_APPS:
	
	INSTALLED_APPS = (
    	'django.contrib.auth',
    	'django.contrib.contenttypes',
    	'django.contrib.sessions',
    	'django.contrib.sites',
    	'django.contrib.admin',
    	'project.app',
	)
	
go to app and add models in models.py

	from django.db import models
	from project.settings import sae_environ
	class Data(models.Models):
		file_name = models.CharField(max_length=100)
		file_data = models.FileField(upload_to='file')
		if sae_environ:
			from project.ready4sae.store import SaeStorage
			file_data = models.FileField(storage=SaeStorage(domain='your_domain',app='your_app'))

then you can use python manage.py syncdb to generate the datebase and the tables

use phpmyadmin to check the database your generated and export it to sql file

now,you can do your views coding.

run the server in local to test by using

	dev_server.py --mysql=username:password:mysql_host:mysql_port --storage-path=/your/path/to/storage/your_domain/your_app

your must make directory first if you will use the storage sevice for your project

if it runs okey,your can now deploy your project to the sae cloud.

change your index.wsgi file to:

	import sae
	import os
	import sys
	import django.core.handlers.wsgi

	os.environ['DJANGO_SETTINGS_MODULE'] = 'project.settings'

	application = sae.create_wsgi_app(django.core.handlers.wsgi.WSGIHandler())

log in to your sae account and go to your app,something must do:

* init your mysql and import from your sql file
* init your storage 

over.



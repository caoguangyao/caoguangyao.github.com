---
layout: post
title: "os x lion mysql python"
date: 2012-08-16 22:11
comments: true
categories: 
---

1.after install mysql on mac.pls be sure the version of mysql.

	file /usr/local/mysql/bin/mysql
	
it may be show
	
	/usr/local/mysql/bin/mysql: Mach-O 64-bit executable x86_64
	
2.download mysql-python package.

	wget http://download.sourceforge.net/sourceforge/mysql-python/MySQL-python-1.2.3.tar.gz
	
	tar zxvf MySQL-python-1.2.3.tar.gz
	cd MySQL-python1.2.3

3.change the site.cfg in line 13

	#mysql_config = /usr/local/bin/mysql_config
	
to:

	mysql_config = /usr/local/mysql/bin/mysql_config

4.build and install from source.

	sudo ARCHFLAGS='-arch x86_64' python setup.py build
	sudo ARCHFLAGS='-arch x86_64' python setup.py install
	
if mysql is a version of i386,use the command:

	sudo ARCHFLAGS='-arch i386' python setup.py build
	sudo ARCHFLAGS='-arch i386' python setup.py install
5.go to python shell

	import MySQLdb
if raise error:

	ImportError: dlopen(/Users/cgy8888/.python-eggs/MySQL_python-1.2.3-py2.6-macosx-10.7-x86_64.egg-tmp/_mysql.so, 2): Library not loaded: libmysqlclient.18.dylib
	Referenced from: /Users/cgy8888/.python-eggs/MySQL_python-1.2.3-py2.6-macosx-10.7-x86_64.egg-tmp/_mysql.so
	Reason: image not found
	
fixed it by doing the following:

	sudo ln -s /usr/local/mysql/lib/libmysqlclient.18.dylib /usr/lib/libmysqlclient.18.dylib
	sudo ln -s /usr/local/mysql/lib /usr/local/mysql/lib/mysql
	
now go to django project and run:
	
	python manage.py syncdb
	
it will work now.

---
layout: post
title: "Django connect to mssql"
date: 2012-11-16 12:53
comments: true
categories: 
---

platform:

	Client:
	ubuntu10.10
	django1.2.7
	django-pyodbc from trunk
	
	Database Server
	ms sql server 2005
	winxp 32bit
	
install tools:
	
	sudo apt-get install unixodbc unixodbc-dev freetds-dev tdsodbc

then modify the config files

	sudo vim /etc/odbcinst.ini
	
	[FreeTDS]
	Description=TDS driver
	Driver=/usr/lib/odbc/libtdsodbc.so
	Setup=/usr/lib/odbc/libtdsS.so
	CPTimeout=
	CPReuse=

odbc.ini:

	sudo vim /etc/odbc.ini 	
	
	[mydsn]
	Description= my dns
	Driver=FreeTDS
	Database=mydb
	Servername=myServer
	TDS_Version=8.0
	Port=1433

freetds.conf:
append to the last.
	
	sudo vim /etc/freetds/freetds.conf

	[myServer]
	host = 192.168.3.128
	port = 1433
	tds version = 8.0
	
config finished

use tsql command to test connect:
	
	tsql -S server_ip_address -U username -P password
		
install pyodbc

	sudo easy_install pyodbc

if have error for gcc:

	sudo apt-get install python-dev build-essential

and try it angain.

write a python script to test:

	import pyodbc

	conn = pyodbc.connect("DSN=mydsn;UID=username;PWD=password")

	cur = conn.cursor()
	cur.execute('select * from test_table')
	for row in cur:
    	print 8 ,row
if have error in conn,use following code test first.

	conn = pyodbc.connect("DRIVER={FreeTDS};Server=server_ip_address;UID=username;PWD=password;DATABASE=dbname")

if it don't run nomarlly ,please check your config file:freetds.conf,odbc.ini,odbcinst.ini

then install django-odbc from trunk
	
	svn co http://django-pyodbc.googlecode.com/svn/trunk/
	
change the settings.py file like the document said.

then your can run syncdb and runserver.

####troubles####
I did use django1.1 and django-pyodbc1.0.x at first,pyodbc is work right,but can't run the django syncdb and runserver.
it throws some error about settings.Time_Zone,and I still don't know how to deal with it.

it seens have some problem use django-pyodbc from trunk too.it happens when syncdb for admin site:throws a None type can't itera.

all settings are done,you can write a model to test it .


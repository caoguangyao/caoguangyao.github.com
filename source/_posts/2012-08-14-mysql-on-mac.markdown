---
layout: post
title: "mysql-on-mac"
date: 2012-08-14 20:56
comments: true
categories: 
---

1.download mysql from mysql website.it's a dmg file,
open the dmg file 4 files in it.install the pkg file and the prefPane file

2.to use mysql command in the terminal,modify the /etc/bashrc file

	#mysql
	alias mysql='/usr/local/mysql/bin/mysql'
	alias mysqladmin='/usr/local/mysql/bin/mysqladmin'
	
add in the end of bashrc file.

3.reopen the terminal,try use mysql command.

4.use mysqladmin to set root user

	mysqladmin -u root password mysqlpassword 

 
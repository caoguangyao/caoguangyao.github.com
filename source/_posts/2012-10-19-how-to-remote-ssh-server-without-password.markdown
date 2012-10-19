---
layout: post
title: "how to remote ssh server without password"
date: 2012-10-19 08:31
comments: true
categories: 
---

1.create ras key by using ssh-keygen

	ssh-keygen -t rsa
this operation will generate two keys in your system.
	
2.copy it to your ssh server
	
	cat ~/.ssh/id_rsa.pub |ssh username@ssh_server_host 'cat >> ~/.ssh/authorized_keys'

3.now you can remote to your server without password.
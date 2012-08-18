---
layout: post
title: "django use filefield in SAE"
date: 2012-08-18 16:17
comments: true
categories: 
---

sina 有个云平台叫sina app enginee，当然，是山寨google app enginee的。虽然这个东西还在alpha阶段，但是对于我等叼丝程序员来说，免去了购买vps麻烦，免费的总归是好的，即使有点问题，也总比没有的强吧。

最近研究了好几天的sae的storage。总觉得很不爽，因为官方api给的很笼统，看不明白，网上demo不多，几乎都同一个人转载，当然也有牛人，不过没找到牛人的源码，所以只好自己推敲了。

再这里要感谢下falcon同学，自己写了一个ready4sae的storage方法，在我纠结了好久以后，用了一下，发现竟然能够用，真是大喜啊.

下面就先来说说如何使用吧：

1.先建立一个store.py的文件：

	#! coding:utf-8

import itertools
import sae.storage
from datetime import datetime
from os.path import basename,splitext
from sae.const import APP_NAME
from django.conf import settings
from django.core.files.storage import Storage
from django.utils.http import urlquote

class SaeStorage(Storage):
    """
    SAE storage
    """

    def __init__(self, domain=None, app=None):
        if app is None:
            app = APP_NAME
        if domain is None:
            domain = settings.SAE_STORAGE_DOMAIN
        self.domain = domain
        self.base_url = "http://%s-%s.stor.sinaapp.com/" % (app,self.domain)
        self.client = sae.storage.Client()
        
    def _open(self, name, mode='rb'):
        """
        not allow in sae, or maybe get the object content and store in memery using StringIO is a good idea. 
        """
        raise sae.storage.PermissionDeniedError('not allow to do this')

    def _save(self, name, content):
        name = self.get_available_name(name)
        data = ''.join(content.chunks())
        ob = sae.storage.Object(data)        
        return self.client.put(self.domain, name, ob)
        
    def delete(self, name):
        return self.delete(self.domain, name)
        
    def exists(self, name):
        return name in [ob['name'] for ob in self.client.list(self.domain)]

    def listdir(self, path):        
        return [ob['name'] for ob in self.client.list(self.domain)]

    def path(self, name):        
      	return self.url

    def size(self, name):
        ob = self.clien.stat(self.domain,name)
        return ob['length']

    def url(self, name):    	
        url = self.client.url(self.domain,name)
        return url.replace(urlquote(self.base_url),'')

    def accessed_time(self, name):
        return self.created_time()    

    def created_time(self, name):
        ob = self.clien.stat(self.domain,name)
        return datetime.fromtimestamp(ob['datetime'])

    def modified_time(self, name):
        return self.created_time()    
        
    def get_available_name(self, name):
        count = itertools.count(1)
        while name in (ob['name'] for ob in self.client.list(self.domain)):
            file_root,file_ext = splitext(basename(name))
            name = '%s_%s%s' % (file_root,count.next(),file_ext)
        return name

2.然后在settings.py 里面写好sae mysql 的设置代码，这些网上请自己搜索。

3.建立自己的models.py

	from django.db import models
	from store import SaeStorage
	class Persion(models.Model):
    	name = models.CharField(max_length=30)
		photo=models.FileField(
	storage=SaeStorage(domain='your_storage_domain',app='your_app_name'))
	#    photo = models.FileField(upload_to='xxx')
	
这里注意下的说，在本地syncdb的时候，要使用默认的storage，不然会报错，但是用dev_server.py的时候是没关系的。


4.就这么简单，本地测试通过的话记得上传到svn，记得把mysql的tables导入到sae先。

5.ps.在此感谢
http://blog.csdn.net/vesslan1029/article/details/7594545
和falcon同学提供的思路。
以上。
	
	
---
layout: post
title: "django_create_inmemoryfile"
date: 2012-10-18 12:51
comments: true
categories: 
---
say we have a model like that:
	
	class Data(models.Model):
		file_name = models.CharField(max_length=100)
		upload_file = models.FileField(upload_to='data')

we what create a file in back,but not save to filesystem.
so use StringIO create string first

	from cStringIO import StringIO
	
	my_str = 'qwer\nasdf\m\n'
	
	buff = StringIO(my_str)
	
	buff.seek(0,2)

use InMemoryUploadFile to create the string file

	from django.core.files.uploadedfile import InMemoryUploadedFile
	from app.models import Data
	file_name = 'myfile.txt'
	file_data = InMemoryUploadedFile(buf,'file',file_name,None,buff.tell(),None)
	
	d = Data(file_name = file_name)
	d.upload_file.save(file_name,file_data)
	d.save() 
	
untile now we create an InMemoryUploadedFile object and save it to our model.

by the way we can use simpler way to create the file

	from django.core.files.base import ContentFile
	
	file_data = ContentFile(buff.getvalue())


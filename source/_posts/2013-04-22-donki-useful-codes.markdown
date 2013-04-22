---
layout: post
title: "donki-useful-codes"
date: 2013-04-22 09:56
comments: true
categories: 
---
## 1. js & jquery

* reverse:

	jQuery.fn.reverse = [].reverse;
    
* object exists:

		jQuery.fn.exists = function(){
        	return this.length>0;
        }
* string format:
        
    	if (!String.prototype.format) {
        	String.prototype.format = function() {
            	var args = arguments;
            	return this.replace(/{(\d+)}/g, function(match, number) { 
              		return typeof args[number] != 'undefined'? args[number]: match;
            	});
          	};
        }
        
plugins > *fileDownlad.js* 

[download](https://github.com/johnculviner/jquery.fileDownload)
## 2. django
* settings.py
	* current directory
            
			import os
    		APP_ROOT = os.path.dirname(__file__)
            
* models
	* dynamic file upload path

			class THistory(models.Model):
            	shop = models.ForeignKey('MShop')
            	section = models.ForeignKey('MSection')
            	revision = models.IntegerField()
            
            	def get_upload_to(self,filename):
                	ext = filename.split('.')[-1]
                	filename = u'%s.%s' % (get_datetime_string(),ext)
                	return os.path.join(self.default_upload_to,filename)
            
            	filepath = models.FileField(upload_to=get_upload_to)
            	user = models.ForeignKey('MUser',null=True,on_delete=models.SET_NULL)
            	upload_date = models.DateTimeField(auto_now=True,auto_now_add=True)
            
            	default_upload_to = 'excelfile'
            
* custom django user (method 1):

        class MUser(User):
            objects = UserManager()
            display_name = models.CharField(max_length = 24, default='')
            permission_choices = (
                    ('1','admin'),
                    ('2','normal'),
                    )
            permission= models.CharField(max_length= 5,choices=permission_choices)
            shop = models.ForeignKey('MShop',blank=True,null=True)
            
            def __unicode__(self):
                return self.usernam

## 3. views

* make download response
        
		def generate_download_response(file_path,file_name):
            response = HttpResponse(readFile(file_path))
            response['Content-Length'] = os.path.getsize(file_path)
            response['Content-type'] = 'application/vnd.ms-excel'
            response['Content-Disposition'] = 'attachment; filename="%s"' % file_name.encode('cp932')
            response.set_cookie('fileDownload','true',path='/')

            return respons

        def readFile(fn, buf_size=262144):
            f = open(fn, "rb")
            while True:
                c = f.read(buf_size)
                if c:
                    yield c
                else:
                    break
            f.close()
        
* custom 404,500 view

	* A).config in settings.py ->debug = False
            
	* B).config in urls.py ->
	handler404  = 'webapp.views.handle_404_view'
    handler500 = 'webapp.views.handler_500_view
            
	* C).config in views.py ->
                
			def handle_500_view(request):
    			t = loader.get_template('500.html')
        		type,value,tb = sys.exc_info()
        		return HttpResponseServerError(t.render(RequestContext(request,{'excption_value':value}))) 
    		
    		def handle_404_view(request):
         		
         		return render(request,'404.html',status=404
         		
* formset
	
	[see detail in django documents](https://docs.djangoproject.com/en/1.4/topics/forms/formsets/)

* transactions
    
    [see detail in django documents](https://docs.djangoproject.com/en/1.4/topics/db/transactions/)
    
* commands

	* start project
	
			django-admin startproject projectname
	* start application

			django-admin startapp appname
	* syncdb  
	
			python manage.py syncdb
	* inspectdb  
	
			python manage.py inspectdb > filename.py

	* run standalone

 			python manage.py runserver 0.0.0.0:portnumber

	* run shell 

			python manage.py shell
	
	* multipy database

			python manage.py syncdb --database=databasename

* static file serve

	* config in settings.py 

   			MEDIA_ROOT = os.path.join(APP_ROOT,'..\\donki-static') 
    		MEDIA_URL = '/excel/
            
 	* config in urls.py
 
 			url(r'^excel/(?P<path>.*)$', django.views.static.serve', {'document_root':MEDIA_ROOT})
    
* apache host
	* wsgi
    
    	[sell detail dowcuments](https://code.google.com/p/modwsgi/)

	* sample virtualhost:
            
          	<VirtualHost *:80>
            ServerName donki.localhost

            WSGIScriptAlias / c:\sourcecode\donki\donki\wsgi.py

            <Directory c:\sourcecode\donki\donki>
            <Files wsgi.py>
            Order deny,allow
            Allow from all
            </Files>
            </Directory>

            Alias /static/ "c:/sourcecode/donki/webapp/static/"

            <Directory "c:/sourcecode/donki/webapp/static/">
            Order deny,allow
            Allow from all
            </Directory>
            ErrorLog "c:\sourcecode\donki\logs\error.log"

            CustomLog "c:\sourcecode\donki\logs\access.log" common
            </VirtualHost>
        
	* command:
   	
   			httpd -t check #script syntax
    		httpd -k restart #restart appache with information            

## 3. python
	* libs
        * MySQL-python
        * cx_Oracle
        * pywin32
    
    * 3.x &2.x

## 4. office vba
* convert column to number

		def conver_column_to_number(col_name):
            return reduce(lambda s,a:s*26+ord(a)-ord('A')+1,col_name,0)
    
* quick deal with cells
 
       range = sheet.Range('start : end')
       details see in win32.py
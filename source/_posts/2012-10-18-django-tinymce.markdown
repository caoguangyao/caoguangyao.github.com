---
layout: post
title: "django_tinymce"
date: 2012-10-18 15:56
comments: true
categories: 
---

Install tiny-mce
	
	sodu easy_install django-tinymce

#####How to use#####

add app in settings.py

	INSTALLED_APPS = (
	…
	'tinymce'
	…
	)

if you want replace textarea tag as html editor in admin page, modify your modeladmin like this

	from tinymce.widgets import TinyMCE
		formfield_overrides = {
			models.TextField:{'widget':TinyMCE(attrs={'cols':80,'rows':20},)},
		}
		
		class Media:
			js = (
				'/d-media/js/tiny_mce/tiny_mce.js',
				'/d-media/js/tiny_mce/textareas.js',
			)
		class Meta:
			model = Article
		…
copy the textareas.js to your static media directory.

more adavnce settings in [here](http://www.tinymce.com/wiki.php)
	
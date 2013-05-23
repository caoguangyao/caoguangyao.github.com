---
layout: post
title: "Java slim3 on Google App Engine"
date: 2013-05-23 12:31
comments: true
categories: 
---
---------------
### **Envirements** ###

install Java

install eclipse

install Google Plugin for Eclipse in 

<code> http://dl.google.com/eclipse/plugin/x.x </code>

note : "x.x" due to  your eclipse version.


install slim3 for Eclipse in 

<code> http://slim3.googlecode.com/svn/updates/ </code>

---------------


### **Configuration** ###

set Eclipse for testing

Preference > Java > Code Style > Organize Imports set "1" to "Number of static imports needed for."

Preference > Java > Editor > Content Assist > Favorites add

<code>org.hamcrest.CoreMatchers</code>

<code>org.junit.Assert</code>

<code>org.junit.matchers.JUnitMatchers</code>

Preference > General > Workspace:

check "Refresh automatically" (eclipse 3.7 above does not have this option)

----------------


### ** Create new simple project ** ###

in eclipse New > project > Slim3 > Silm3 Project > next > give a project name and root package name (should be same)

open build.xml in your project 

right click menu Run as > Ant Build (first item)

will popup a dialog,then input "/twitter/" ("twitter/" is short for "/twitter/index") click "ok" button.

"twitter" is the controller name,
so it will automatically create controller class:

projectname.controller.twitter.IndexController

projectname.controller.twitter.IndexControllerTest

war/twitter/index.jsp

Now,you can run testcase for IndexControllerTest by right menu > Run as > JUnit Test.
It will show a green progressbar for no problem.

You can run the project by right menu > Run as > Web Application.

Open browser with "http://localhost:8888/twitter/",and see what happend

----------------

### **Add model** ###


Right menu build.xml > Run as > Ant Build (second item ) > gen-model 

will popup a dialog to input a model name. let's name it "Tweet".

then will automatic add a class Tweet.java

**Add field to model**
In Tweet.java add 2 fields 

	private String content;
	private Date createDate = new Date();

and add the getter and setter for class.

**Add service for model**

Right click build.xml > Run as > Ant Build(second item) > gen-model > in shown popup dialog, give a name with "TweetService",then will create an empty service class file named TweetService.java.

Now,change this class:

	…
	
	import projectname.model.Tweet;
	
	public class TweeterService{
		public Tweet tweet(Map(String,Object) input){
			Tweet tweet = new Tweet();
			BeanUtil.copy(input,tweet);
			Transaction tx = Datastore.beginTransaction();
			Datastore.put(tweet);
			tx.commit()
			return tweet;
		}
	}
	
note:

BeanUtil copies property values from the input data to the model for all cases where the property names are the same. Even if the property type of the input is different from the one of the model, the value is converted appropriately.
To put a model to the datastore, you use Datastore.put(Object model).
See Creating a model.
If an exception occurred during the transaction, it is rolled back by Slim3 automatically.

**Create getList method**

add code to the class:

	…
	public class TwitterService {
	
		private TweetMeta t = new TweetMeta();
	
		…
		public List<Tweet> getTweetList() {
        	return Datastore.query(t).sort(t.createdDate.desc).asList();
    	}
	} 

----------------

### **Using model in controller** ###

Go to IndexController.java,change some code to

	…
	import projectname.model.Tweet;
	import projectname.service.TwitterService;
	
	public class IndexController extends Controller {

    	private TwitterService service = new TwitterService();

    	@Override
    	public Navigation run() throws Exception {
        	List<Tweet> tweetList = service.getTweetList();
        	requestScope("tweetList", tweetList);
        	return forward("index.jsp");
    	}
	}

note:requestScope("tweetList", tweetList) is short for request.setAttribute("tweetList", tweetList)


----------------

### **Render to template** ###

To render to template,just change the index.jsp file

	<%@page pageEncoding="UTF-8" isELIgnored="false"%>
	<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
	<%@taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt"%>
	<%@taglib prefix="fn" uri="http://java.sun.com/jsp/jstl/functions"%>
	<%@taglib prefix="f" uri="http://www.slim3.org/functions"%>

	<html>
	<head>
	<meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
	<title>twitter Index</title>
	<link rel="stylesheet" type="text/css" href="/css/global.css" />
	</head>
	<body>
	<p>What are you doing?</p>
	
	<c:forEach var="e" items="${tweetList}">
	${f:h(e.content)}
	<hr />
	</c:forEach>
	</body>
	</html>

note:I list tweeted contents using the tweetList variable.
f:h is a JSP function to escape HTML.

*Now if you have data in the tweetList,the page will show all the data.content in the list.*

-----------------

#### To be Continue... ####
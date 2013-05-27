---
layout: post
title: "Slim3 datastore and controller"
date: 2013-05-27 09:37
comments: true
categories: 
---

### Controller ###

---

interface: run()

URL mapping:mapping rule as follows:

**normal usage**

path | controller
---|---
/|packagename.controller.IndexController
/xxx|packagename.controller.XxxController
/xxx/|packagename.controller.xxx.IndexController
/xxx/yyy|packagename.controller.xxx.YyyController
/xxx/yyy/zzz|packagename.controller.xxx.yyy.ZzzController

note: packagename is defined in web.xml

if you are use GWT add "server" after packagename

** url rewrite **

Define packagename.controller.AppRouter first.

for example:
	
	package slim3.demo.controller;

	import org.slim3.controller.router.RouterImpl;

	public class AppRouter extends RouterImpl {

    	public AppRouter() {
        	addRouting("/_ah/mail/{address}", "/mail/receive?address={address}");
        	addRouting(
            	"/{app}/edit/{key}/{version}",
            	"/{app}/edit?key={key}&version={version}");
        	addRouting(
            	"/{app}/delete/{key}/{version}",
            	"/{app}/delete?key={key}&version={version}");
                	addRouting(
            	"/{app}/find/*path",
            	"/{app}/find?path={path}");
    		}	
	} 
	
The request for /_ah/mail/xxx@gmail.com is routed to /mail/receive?address=xxx@gmail.com.

The request for /blog/edit/xxx/1 is routed to /blog/edit?key=xxx&version=1.

The request for /blog/delete/xxx/1 is routed to /blog/delete?key=xxx&version=1.

The request for /blog/find/xxx/yyy/zzz is routed to /blog/find?path=xxx%2Fyyy%2Fzzz, (xxx/yyy/zzz gets URL-encoded).

A place holder({xxx}) does not match "/", because "/" is a path separator.

A catch-all place holder(*xxx) must be at the end of the match (From) string and matches everything including "/" up to the end. 

** file upload **

The uploaded result is stored in FileItem.java. So you can get it as follows:

	FileItem formFile = requestScope("formFile");

The size of uploaded data is limited to 10MB. 
The maximum entity size is 1MB. so:
UploadedDataFragment:

	@Model
	public class UploadedData {

    	@Attribute(primaryKey = true)
    	private Key key;

    	@Attribute(version = true)
    	private Long version = 0L;

    	private String fileName;

    	private int length;

    	@Attribute(persistent = false)
    	private InverseModelListRef<UploadedDataFragment, UploadedData> fragmentListRef =
        	new InverseModelListRef<UploadedDataFragment, UploadedData>(
            	UploadedDataFragment.class,
            	"uploadDataRef",
            	this,
            	new Sort("index"));
    		...
	}
UploadedDataFragment

	@Model
	public class UploadedDataFragment {

    	@Attribute(primaryKey = true)
    	private Key key;

    	@Attribute(lob = true)
    	private byte[] bytes;

    	private ModelRef<UploadedData> uploadDataRef =
        	new ModelRef<UploadedData>(
            	UploadedData.class);

    	private int index;

    	...
	}
UploadService

	public UploadedData upload(FileItem formFile) {
    	if (formFile == null) {
        	return null;
    	}
    	List<Object> models = new ArrayList<Object>();
    	UploadedData data = new UploadedData();
    	models.add(data);
    	data.setKey(Datastore.allocateId(d));
    	data.setFileName(formFile.getShortFileName());
    	data.setLength(formFile.getData().length);
    	byte[] bytes = formFile.getData();
    	byte[][] bytesArray = ByteUtil.split(bytes, FRAGMENT_SIZE);
    	Iterator<Key> keys =
        	Datastore.allocateIds(data.getKey(), f, bytesArray.length).iterator();
    	for (int i = 0; i < bytesArray.length; i++) {
        	byte[] fragmentData = bytesArray[i];
        	UploadedDataFragment fragment = new UploadedDataFragment();
        	models.add(fragment);
        	fragment.setKey(keys.next());
        	fragment.setBytes(fragmentData);
        	fragment.setIndex(i);
        	fragment.getUploadDataRef().setModel(data);
    	}
    	Transaction tx = Datastore.beginTransaction();
    	for (Object model : models) {
        	Datastore.put(tx, model);
    	}
    	tx.commit();
    	return data;
	}

** validation **

Slim3 provides a simple validation framework as follows:

	Validators v = new Validators(request);
	v.add("arg1", v.required(), v.integerType());
	v.add("arg2", v.required(), v.integerType(), v.longRange(3, 5));
	if (v.validate()) {
    	//OK
	} else {
    	//NG
    	Errors errors = v.getErrors();
    	System.out.println(errors.get("arg1"));
    	System.out.println(errors.get("arg2"));
	}

The error messages are stored in <project>/src/application[_locale].properties as follows:

	validator.required={0} is required.
	validator.byteType={0} must be a byte.
	validator.shortType={0} must be a short.
	validator.integerType={0} must be an integer.
	validator.longType={0} must be a long.
	validator.floatType={0} must be a float.
	validator.doubleType={0} must be a double.
	validator.numberType={0} is not a number({1}).
	validator.dateType={0} is not a date({1}).
	validator.minlength={0} can not be less than {1} characters.
	validator.maxlength={0} can not be greater than {1} characters.
	validator.range={0} is not in the {1} to {2} range.
	validator.regexp={0} is invalid.
	
{0} is an attribute name. If you want to change the attribute name by locale, define label.<attribute name> entry in <project>/src/application[_locale].properties.
	
	label.arg1=xxx
	label.arg2=yyy
	
If the attribute is stored to a model, you can specify it type-safely using Meta data of model:

	Validators v = new Validators(request);
	BlogMeta meta = BlogMeta.get();
	v.add(meta.title, v.required());
	v.add(meta.content, v.required());
	
## Models ##
---
To declare a Java class as model, give the class a @Model annotation. For example:

	import org.slim3.datastore.Model;

	@Model
	public class Employee {
    	// ...
	}
	
Fields of the data class are stored in the datastore. A persistent field must have getter and setter methods.

	import java.util.Date;

	// ...
    	private Date hireDate;

    	public Date getHireDate() {
        	return hireDate;
    	}

    	public void setHireDate(Date hireDate) {
        	this.hireDate = hireDate;
    	}
   
A data class must have one field dedicated to storing the primary key of the corresponding datastore entity. A key field must be com.google.appengine.api.datastore.Key. A key use a @Attribute(primaryKey = true) annotation:

	import com.google.appengine.api.datastore.Key;
	import org.slim3.datastore.Attribute;

	// ...
    	@Attribute(primaryKey = true)
    	private Key key;

	// ... accessors ...  
	
**Serializable Objects**

You can store an instance of a Serializable class as a Blob value. To tell Slim3 Datastore to serialize the value, the field uses the annotation @Attribute(lob = true). Blob values are not indexed and cannot be used in query filters or sort orders.

Here is an example of a simple Serializable class that represents a file, including the file contents, a filename and a MIME type.

	import java.io.Serializable;

	public class DownloadableFile implements Serializable {
    	private static final long serialVersionUID = 1L;
    	private byte[] content;
    	private String filename;
    	private String mimeType;

    	// ... accessors ...
	}

To store an instance of a Serializable class as a Blob value in a property, declare a field whose type is the class, and use the @Attribute(lob = true) annotation:

	import DownloadableFile;
	import org.slim3.datastore.Attribute;
	// ...
    	@Attribute(lob = true)
    	private DownloadableFile file;


**Creating a model**

To store a model in the datastore, you call Datastore.put() method, passing it the instance.

	Employee employee = new Employee();
	...
	Datastore.put(employee);
**Getting a model by Key**

	Key key = ...;
	Employee emp = Datastore.get(Employee.class, key);
**Updating a model**

To update a model, fetch the model, modify it, and put it to the datastore:

	public void updateEmployeeTitle(Key key, String newTitle) {
    	Employee emp = Datastore.get(Employee.class, key);
    	emp.setTitle(newTitle);
    	Datastore.put(emp);
	}
**Deleting a model**

To delete a model from the datastore, call Datastore.delete() method with the key:

	Key key = ...;
	Datastore.delete(key);

**Query**

A filter specifies a field name, an operator, and a value. The value must be provided by the app; it cannot refer to another property, or be calculated in terms of other properties. The operator can be any of the following: "equal", "lessThan", "lessThanOrEqual", "greaterThan", "greaterThanOrEqual", "isNotNll", "startsWith", "notEqual", "in". An entity must match all filters to be a result. The logical combination of filters is "and".

The "notEqual" operator actually performs 2 queries: one where all other filters are the same and the "notEqual" filter is replaced with a "lessThan" filter, and one where the "notEqual" filter is replaced with a "greaterThan" filter. The results are merged, in order. As described below in the discussion of inequality filters, a query can only have one "notEqual" filter, and such a query cannot have other inequality filters.

The "in" operator also performs multiple queries, one for each item in the provided list value where all other filters are the same and the "in" filter is replaced with an equal filter. The results are merged, in the order of the items in the list. If a query has more than 1 "in" filter, the query is performed as multiple queries, one for each combination of values in the "in" filters.

A single query containing "notEqual" or "in" operators is limited to 30 sub-queries.

	Date hireDate = ...;
	EmployeeMeta e = EmployeeMeta.get();
	List<Employee> list = Datastore.query(e)
    	.filter(e.lastName.equal("Smith"), e.hireDate.greaterThan(hireDate))
    	.asList();

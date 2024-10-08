## Data persistence using Voyage and MongoDB


So far we've used objects stored in memory for our model, which is fine because saving the Pharo image also saves the objects. Nevertheless, it would be better to save these objects (blog posts) into an external database. Pharo supports multiple object serializers such as Fuel (binary format) or STON (text format). These serializers are useful and powerful. Often with a single line of code, we can save a full graph of objects, as explained in the Enterprise Pharo book (available at [http://books.pharo.org](http://books.pharo.org)).

In this chapter, we will use another option: saving data in a _document database_ such as MongoDB ([https://www.mongodb.com](https://www.mongodb.com)) using the Voyage framework. Voyage provides a unified API to store and retrieve objects in various document-based databases such as MongoDB or UnQLite. But to start with we will use Voyage's ability to simulate an external database in memory. This is really useful during development. You can then install a local MongoDB database and access it through Voyage afterwards if you like. As you will see, this second step will have little impact on our code.

The last chapter explains how to load the code of previous chapters if needed.

### Configure Voyage to save TBBlog objects


By defining the class method `isVoyageRoot`, and having it return `true`, we declare that objects of this class must be saved into the database as _root objects_. It means that the database will contain as many documents as instances of this class:

```
TBBlog class >> isVoyageRoot
   "Indicates that instances of this class are top level documents in noSQL databases"
   ^ true
```


We should now establish a connection to a real database or work in memory. Let's start off by working in memory by using this expression:

```
VOMemoryRepository new enableSingleton
```


The `enableSingleton` message indicates to Voyage that we will only use one database. This will free us to specify the database each time. We create and initialize the database in memory in a class-side method on `TBBlog` named `initializeVoyageOnMemoryDB`:

```
TBBlog class >> initializeVoyageOnMemoryDB

   VOMemoryRepository new enableSingleton
```


The `reset` class method re-initializes the database. The `initialize` class method ensures that the database is initialized when we load TinyBlog's code. Do not forget to execute the expression `TBBlog initialize` to ensure that the database is initialized.

```
TBBlog class >> reset

      self initializeVoyageOnMemoryDB
```


```
TBBlog class >> initialize

      self reset
```


The class-side `current` method is trickier. Before using Voyage, we implemented a simple singleton pattern (`TBBlog current`). However, it does not work anymore. Imagine we save our blog and then the server crashed, or that we reload a new version of the code: it would re-initialize the connection and create a new, fresh instance of the blog. It would then be possible to end up with a different instance than the saved one.

So we must change the implementation of the `current` class method to make a database request and retrieve the saved objects. Since we only save _one_ blog object, we just need to get that one blog object out of the collection of objects. Which is a simple case of `self selectOne: [ :each | true ]` or even just `self selectAll anyOne`. But if the database contains no instance, we create a new one and save it. So we end up with a method that looks something like this:

```
TBBlog class >> current

   ^ self selectAll 
			ifNotEmpty: [ :x | x anyOne ]
			ifEmpty: [ self new save ]
```


We can also remove the class instance variable named `uniqueInstance` that we previously used to store our singleton object:

```
TBBlog class

	instanceVariableNames: ''
```


### Saving a blog


Each time we modify a blog object, we must propagate changes into the database. For example, we modify the `writeBlogPost:` method to save the blog when we add a new post|

```
TBBlog >> writeBlogPost: aPost
	"Write the blog post in database"
	posts add: aPost.
	self save
```


We also save the blog when removing posts from a blog:

```
TBBlog >> removeAllPosts
	posts := OrderedCollection new.
	self save
```


### Revising unit tests


We now save blogs in a database, either in memory or in an external MongoDB server, through Voyage. We must be careful with unit tests that modify the database because they may corrupt production data. To circumvent this dangerous situation, a test should not modify the state of the system. 

To solve this situation, before running a test we will keep a reference to the current blog and create a new context and restore it after test execution.

Let's add an instance variable `previousRepository` in the `TBBLogTest` class.

```
TestCase subclass: #TBBlogTest
	instanceVariableNames: 'blog post first previousRepository'
	classVariableNames: ''
	package: 'TinyBlog-Tests'
```


Then, we modify the `setUp` method to save the database before each test execution.
We create a temporary database object that will be used by the test.

```
TBBlogTest >> setUp

    super setUp.
	previousRepository := VORepository current.
	VORepository setRepository: VOMemoryRepository new.
	blog := TBBlog current.
	first := TBPost title: 'A title' text: 'A text' category: 'First Category'.
	blog writeBlogPost: first.
	post := (TBPost title: 'Another title' text: 'Another text' category: 'Second Category') beVisible
```


In the `tearDown` method executed after each test, we restore the original database object:

```
TBBlogTest >> tearDown

	VORepository setRepository: previousRepository.
    super tearDown
```


### Querying the database


The database is currently in memory and we can access to the blog object using the `current` class-side method of the `TBBlog` class.
It is enough to show the API of Voyage since it will be the same to access a real MongoDB database.

You can create posts:
```
TBBlog createDemoPosts
```


You can count the number of blog saved. `count` is part of the Voyage API. In this example, we get the result 1 because the blog is implemented as a Singleton.

```
TBBlog count
>1
```


Similarly, you can retrieve all saved root objects of one kind.

```
TBBlog selectAll	
```


You can also remove a root object using the `remove` message.

You can discover more about the Voyage API by looking at:

- the `Class` class,
- the `VORepository` class which is the root of the hierarchy of all databases either in memory or external.


Those queries will be more relevant with more objects but they would be similar.

### Discussion


This section should not be implemented. It is only described as an example (More information about Voyage can be found in the Enterprise Pharo book [http://books.pharo.org](http://books.pharo.org)). We want to illustrate that declaring a class as a Voyage root has an influence on how an instance of this class is saved and reloaded. 

So far, a post (an instance of `TBPost`) is not declared as a Voyage root. Post objects are therefore saved as sub-parts into the blog object they belong to. It implies that a post is not guaranteed to be unique after saving and re-loading from the database. Indeed, after loading each blog object will have its own posts objects even if some posts were shared before saving. Shared objects before saving will be duplicated for each root object after loading.

We can declare posts as root objects meaning that a post can be saved independently from a blog. It implies that saved blogs have a reference to a `TBPost` object. This would preserve post sharing between blog objects. 

However, not all objects should be root objects. If we represent post comments, we would not define them as root objects too because manipulating a comment outside of its context (a post) does not make sense.

#### Post as Root = Uniqueness


If you want to share posts and make them unique between multiple blogs, therefore, the `TBPost` class must be declared as a root in the database. In this case, posts are saved as autonomous entities, and instances of `TBBlog` will reference post entities instead of embedding them. The consequence is that a post is unique and can be shared via reference from a blog. To achieve this, we **would** define the following methods:

```
TBPost class >> isVoyageRoot
   "Indicates that instances of this class are top level documents in noSQL databases"
   ^ true
```


During the addition of a post to a blog, it would be important to save both the blog and the new post.

```
TBBlog >> writeBlogPost: aPost
   "Write the blog post in database"
   posts add: aPost.
   aPost save. 
   self save 
```



```
TBBlog >> removeAllPosts
   posts do: [ :each | each remove ].
   posts := OrderedCollection new.
   self save.
```


In the `removeAllPosts`  method, we first remove all posts, then update the collection and finally save the blog.

### Configure an external MongoDB database [Optional]


By using Voyage, we can easily save our model objects into a MongoDB database. This section explains how to proceed and the few modifications to make to our code. This is not mandatory, and even if you do it, we would encourage you to continue to work with an in-memory database afterwards.

#### Installing MongoDB


Regardless of your operating system (Linux, MacOS or Windows), you can install a local MongoDB server on your machine (see [https://www.mongodb.com](https://www.mongodb.com)). This is useful to test your application without requiring an Internet connection. Instead of directly installing MongoDB, we suggest installing Docker ([https://www.docker.com](https://www.docker.com)) on your machine and executing a MongoDB container using the following command on your command line:

```
	docker run --name mongo -p 27017:27017 -d mongo
```


!!note The running MongoDB server must not use authentication (it is not the case with the default installation) because the new SCRAM authentication mechanism used by MongoDB 3.0 is currently not supported by Voyage.

Some useful Docker commands:

```
	# to stop your MongoDB docker container
	docker stop mongo
	
	# to re-start your container
	docker start mongo
	
	# to delete your container (it must be stopped before)
	docker rm mongo
```



#### Connecting a local MongoDB server


Once installed, you can connect to a MongoDB server directly from Pharo.
We define the method named `initializeLocalhostMongoDB` to establish the connection to the local MongoDB server (localhost, default port) and access the database named 'tinyblog'|

```
TBBlog class >> initializeLocalhostMongoDB
   | repository |
   repository := VOMongoRepository database: 'tinyblog'.
   repository enableSingleton.
```


Reset the class to set a new connection to the database:

```
TBBlog class >> reset
   self initializeLocalhostMongoDB
```


Now, if you recreate demo posts, they are automatically saved into your local MongoDB database:

```
TBBlog reset.
TBBlog createDemoPosts 
```


#### In case of emergency


If you need to re-initialize completely an external database, you can use the `dropDatabase` method:

```
(VOMongoRepository
   host: 'localhost'
   database: 'tinyblog') dropDatabase
```


You can also do it in the command expression when `mongod` is running with: 

```
mongo tinyblog --eval "db.dropDatabase()"
```


or by connecting to the docker container it is running in:

```
docker exec -it mongo bash -c 'mongo tinyblog --eval "db.dropDatabase()"'
```


#### Warning: changing TBBlog definition


When you use an external MongoDB database instead of a memory one, each time you add new root objects or modify the definition of some root objects, it is important to reset the cache maintained by Voyage. It can be done using:

```
VORepository current reset
```


### Conclusion


Voyage provides a nice API to transparently manage the storage of objects either into memory or a document database. Application data is now saved into a database, and so we are ready to build a user interface.

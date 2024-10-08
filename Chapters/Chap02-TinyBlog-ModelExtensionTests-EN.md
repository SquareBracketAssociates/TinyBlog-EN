## TinyBlog: Extending and Testing the Model

@chapModelExtensionAndUnitTests

In this chapter we extend the model and add more tests.
Note that when you will get fluent in Pharo, you will tend to write first your tests and then execute tests to code in the debugger. 
We did not do it because coding in the debugger requires more explanation. 
You can see such a practice in the Mooc video entitled  _Coding a Counter in the Debugger_ (See [http://mooc.pharo.org](http://mooc.pharo.org)) and read the book _Learning Object-Oriented Programming, Design with TDD in Pharo_ ([http://books.pharo.org](http://books.pharo.org)).

Before starting, use back the code of the previous chapter or use the information of Chapter *@cha:loading@*.

%  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
### TBBlog class

%  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We develop the class `TBBlog` that contains posts (as shown by Figure *@postBlogUml@*). We define some unit tests.

![TBBlog: A simple class containing posts. % width=55&label=postBlogUml](figures/postBlogUML.pdf)

Here is its definition: 

```
Object subclass: #TBBlog
   instanceVariableNames: 'posts'
   classVariableNames: ''
   package: 'TinyBlog'
```


We initialize the `posts` instance variable to an empty collection.

```
TBBlog >> initialize
   super initialize.
   posts := OrderedCollection new
```



### Only One Blog Object


In the rest of this project, we assume that we will manage only one blog. 
Later, you may add the possibility to manage multiple blogs such as one per user of the TinyBlog application. 
Currently, we use a Singleton design pattern on the `TBBlog` class. 
However pay attention since this pattern introduces a kind of global variable in your application and brings less modularity to your system. 
Therefore avoid making explicit references to the singleton, better use an instance variable whose value first refers to the singleton so that later you can pass another object without being forced to rewrite everything.
Do not generalize what we are doing for this class.


Since all the management of a singleton is a class behavior, we define such methods at the class level of `TBBlog`. 
We define an instance variable at the class level: 

```
TBBlog class
   instanceVariableNames: 'uniqueInstance'
```


Then we define two methods to manage the singleton. 
```
TBBlog class >> reset
   uniqueInstance := nil
```


```
TBBlog class >> current
   "answer the instance of the TBRepository"
   ^ uniqueInstance ifNil: [ uniqueInstance := self new ]
```


We redefine the class method `initialize` so that when the class is loaded in memory
the singleton got reset.

```
TBBlog class >> initialize
   self reset
```



### Testing the Model


We now adopt a Test-Driven Development approach i.e., we will write a unit test first and then develop the functionality until the test is green. We will repeat this process for each functionality of the model.

We create unit tests in the `TBBlogTest` class that belongs to the `TinyBlog-Tests` tag. A tag is just a label to sort classes inside a package (See menu item 'Add Tag...'). We use a tag because using two packages will make this project more complex. However, while implementing a real application, it is recommended to have one (or multiple) separate test packages.

```
TestCase subclass: #TBBlogTest
   instanceVariableNames: 'blog post first'
   classVariableNames: ''
   package: 'TinyBlog-Tests'
```


Before each test execution, the `setUp` method initializes the test context (also called test fixture).
For example, it erases the blog content, adds one post and creates another temporary post that is not saved.

Pay attention since we will have to modify such behavior in the future else each time we will run the test
we will destroy our data. 
This is an example of the kind of insidious behavior that a singleton introduces.

```
TBBlogTest >> setUp
   blog := TBBlog current.
   blog removeAllPosts.

   first := TBPost title: 'A title' text: 'A text' category: 'First Category'.
   blog writeBlogPost: first.

   post := (TBPost title: 'Another title' text: 'Another text' category: 'Second Category') beVisible
```


As you may have noticed, we test different configurations.
Posts do not belong to the same category, one is visible and the other is not visible.


At the end of each test, the `tearDown`  method is executed and resets the blog.

```
TBBlogTest >> tearDown
   TBBlog reset
```


Here we see one of the limits of using a Singleton. Indeed, if you deploy a blog and then execute the tests, you will lose all posts that have been created because we reset the Blog singleton. We will address this problem in the future.

We will now develop tests first and then implement all functionalities to make them green.

### A First Test


The first test adds a post in the blog and verifies that this post is effectively added.

```
TBBlogTest >> testAddBlogPost
   blog writeBlogPost: post.
   self assert: blog size equals: 2
```


If you try to execute it, you will notice that this test is not green \(does not pass\) because we did not define the methods `writeBlogPost:`, `removeAllPosts`, and `size`.
Let's add them:

```
TBBlog >> removeAllPosts
   posts := OrderedCollection new
```


```
TBBlog >> writeBlogPost: aPost
   "Add the blog post to the list of posts."
   posts add: aPost
```


```
TBBlog >> size
   ^ posts size
```


The previous test should now pass (i.e. be green).

### Increasing Test Coverage


We should also add tests to cover all functionalities that we introduced.

```
TBBlogTest >> testSize
   self assert: blog size equals: 1
```


```
TBBlogTest >> testRemoveAllBlogPosts
   blog removeAllPosts.
   self assert: blog size equals: 0
```


### Other Functionalities


We follow the test-driven way of defining methods: First, we define a test. Then we verify that this test
is failing. Then we define the method under test and finally verify that the test passes.

#### All Posts 


Let's create a test that fails:

```
TBBlogTest >> testAllBlogPosts
   blog writeBlogPost: post.
   self assert: blog allBlogPosts size equals: 2
```


And the model code that makes it succeed:

```
TBBlog >> allBlogPosts
   ^ posts
```


Your test should pass.

#### Visible Posts


We define a new unit test accessing visible blogs:

```
TBBlogTest >> testAllVisibleBlogPosts
   blog writeBlogPost: post.
   self assert: blog allVisibleBlogPosts size equals: 1
```


We add the corresponding method: 

```
TBBlog >> allVisibleBlogPosts
   ^ posts select: [ :p | p isVisible ]
```


Verify that the test passes. 

#### All Posts of a Category

The following test verifies that we can access all the posts of a given category. 
Once defined, we should make sure that the test failed.
```
TBBlogTest >> testAllBlogPostsFromCategory
   self assert: (blog allBlogPostsFromCategory: 'First Category') size equals: 1
```


Then we can define the functionality and make sure that our test passes.

```
TBBlog >> allBlogPostsFromCategory: aCategory
   ^ posts select: [ :p | p category = aCategory ]
```


Verify that the test passes. 
#### All visible Posts of a Category

The following test verifies that we can access all the visible posts of a given category. 
Once defined, we should make sure that the test failed.

```
TBBlogTest >> testAllVisibleBlogPostsFromCategory
   blog writeBlogPost: post.
   self assert: (blog allVisibleBlogPostsFromCategory: 'First Category') size equals: 0.
   self assert: (blog allVisibleBlogPostsFromCategory: 'Second Category') size equals: 1
```


Then we can define the functionality and make sure that our test passes.
```
TBBlog >> allVisibleBlogPostsFromCategory: aCategory
	^ posts select: [ :p | p category = aCategory 
									and: [ p isVisible ] ]
```


Verify that the test passes. 
#### Check unclassified posts


The following test verifies that we do not have unclassified blogs in our test fixture. 

```
TBBlogTest >> testUnclassifiedBlogPosts
   self assert: (blog allBlogPosts select: [ :p | p isUnclassified ]) size equals: 0
```


Verify that the test passes. 

#### Retrieve all categories


Again we define a new test and verify that it fails.
```
TBBlogTest >> testAllCategories
   blog writeBlogPost: post.
   self assert: blog allCategories size equals: 2
```


We then add the new behavior.
```
TBBlog >> allCategories
   ^ (self allBlogPosts collect: [ :p | p category ]) asSet
```


Verify that the test passes. 

### Testing data 


To help you testing the application, you can add the following method that creates multiple posts.

```
TBBlog class >> createDemoPosts
   "TBBlog createDemoPosts"
   self current 
      writeBlogPost: ((TBPost title: 'Welcome in TinyBlog' text: 'TinyBlog is a small blog engine made with Pharo.' category: 'TinyBlog') visible: true);
      writeBlogPost: ((TBPost title: 'Report Pharo Sprint' text: 'Friday, June 12 there was a Pharo sprint / Moose dojo. It was a nice event with more than 15 motivated sprinters. With the help of candies, cakes and chocolate, huge work has been done' category: 'Pharo') visible: true);
      writeBlogPost: ((TBPost title: 'Brick on top of Bloc - Preview' text: 'We are happy to announce the first preview version of Brick, a new widget set created from scratch on top of Bloc. Brick is being developed primarily by Alex Syrel (together with Alain Plantec, Andrei Chis and myself), and the work is sponsored by ESUG. 
      Brick is part of the Glamorous Toolkit effort and will provide the basis for the new versions of the development tools.' category: 'Pharo') visible: true);
      writeBlogPost: ((TBPost title: 'The sad story of unclassified blog posts' text: 'So sad that I can read this.') visible: true);
      writeBlogPost: ((TBPost title: 'Working with Pharo on the Raspberry Pi' text: 'Hardware is getting cheaper and many new small devices like the famous Raspberry Pi provide new computation power that was one once only available on regular desktop computers.' category: 'Pharo') visible: true)
```


If you inspect the result of the following snippet, you will see that the current blog contains 5 posts:

```
	TBBlog createDemoPosts ; current
```


Be aware that if you execute this `createDemoPosts` method multiple times, your blog singleton object will contain multiple copies of these posts.

### Possible Extensions


Many extensions can be made such as: retrieving the list of categories that contain at least one visible post, deleting a category and all posts that it contains, renaming a category, moving a post from one category to another, making (in)visible one category and all its content, etc. We encourage you to develop some of them. 

### Conclusion


You now have the full model of TinyBlog as well as some unit tests. You are now ready to implement more advanced functionality such as  database storage or a first Web front-end. Do not forget to save your code. 

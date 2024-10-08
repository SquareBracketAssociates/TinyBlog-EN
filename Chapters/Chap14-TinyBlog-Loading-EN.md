## Loading Chapter Code

@cha:loading

This chapter contains the expressions to load the code described in each of the chapters.
Such expressions can be executed in any Pharo 8.0 (or above) image.
Nevertheless, using the Pharo MOOC image (cf. Pharo Launcher) is usually faster because it already includes several libraries such as: Seaside, Voyage, ...

When you start for example the chapter 4, you can load all the code of the previous Chapters (1, 2, and 3) by following the process described in the following section 'Chapter 4'.

Obviously, we believe that this is better that you use you own code but having our code at hand can help you in case you would be stuck.

### Chapter 3: Extending and Testing the Model


You can load the correction of the previous chapter as follows:

```
Metacello new
   baseline:'TinyBlog';
   repository: 'github://LucFabresse/TinyBlog:chapter2/src';
   onConflict: [ :ex | ex useLoaded ];
   load
```


Run the tests!
To do so, you can use the TestRunner  (Tools menu > Test Runner), look for the package TinyBlog-Tests, and click on "Run Selected".
All tests should be green.

### Chapter 4: Data Persistency using Voyage and Mongo


You can load the correction of the previous chapter as follows:

```
Metacello new
   baseline:'TinyBlog';
   repository: 'github://LucFabresse/TinyBlog:chapter3/src';
   onConflict: [ :ex | ex useLoaded ];
   load
```


Once loaded execute the tests.

### Chapter 5: First Steps with Seaside


You can load the correction of the previous chapter as follows:

```
Metacello new
   baseline:'TinyBlog';
   repository: 'github://LucFabresse/TinyBlog:chapter4/src';
   onConflict: [ :ex | ex useLoaded ];
   load
```


Execute the tests.

To test the application, start the HTTP server:
```
ZnZincServerAdaptor startOn: 8080.
```


Open your web browser at `http://localhost:8080/TinyBlog`

You may need to recreate some posts as follows:

```
TBBlog reset ; createDemoPosts
```


### Chapitre 6: Web Components for TinyBlog


You can load the correction of the previous chapter as follows:

```
Metacello new
   baseline:'TinyBlog';
   repository: 'github://LucFabresse/TinyBlog:chapter5/src';
   onConflict: [ :ex | ex useLoaded ];
   load
```


Same process as above.

### Chapitre 7: Managing Categories


You can load the correction of the previous chapter as follows:

```
Metacello new
   baseline:'TinyBlog';
   repository: 'github://LucFabresse/TinyBlog:chapter6/src';
   onConflict: [ :ex | ex useLoaded ];
   load
```



To test the application, start the HTTP server:

```
ZnZincServerAdaptor startOn: 8080.
```


Open your web browser at `http://localhost:8080/TinyBlog`


You may need to recreate some posts as follows:

```
TBBlog reset ; createDemoPosts
```




### Chapitre 8: Authentication and Session


You can load the correction of the previous chapter as follows:

```
Metacello new
   baseline:'TinyBlog';
   repository: 'github://LucFabresse/TinyBlog:chapter7/src';
   onConflict: [ :ex | ex useLoaded ];
   load
```


To test the application, start the HTTP server:

```
ZnZincServerAdaptor startOn: 8080.
```


### Chapitre 9: Administration Web Interface and Automatic Form Generation



You can load the correction of the previous chapter as follows:

```
Metacello new
   baseline:'TinyBlog';
   repository: 'github://LucFabresse/TinyBlog:chapter8/src';
   onConflict: [ :ex | ex useLoaded ];
   load
```


### Latest Version of TinyBlog


The most up-to-date version of TinyBlog can be loaded as follows:

```
Metacello new
   baseline:'TinyBlog';
   repository: 'github://LucFabresse/TinyBlog/src';
   onConflict: [ :ex | ex useLoaded ];
   load.
```

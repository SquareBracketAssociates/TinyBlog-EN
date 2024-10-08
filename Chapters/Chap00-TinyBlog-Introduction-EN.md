## About this book

@chapIntro
In this book, we will guide you to develop a mini project: a small web application, named TinyBlog, that manages a blog system (see its final state in Figure *@TinyBlogOnPharoCloudHere@*).
The idea is that a visitor of the website can read the posts and that the post author can connect to the website as admin to manage its posts (add, remove and modify existing ones).

TinyBlog is a small pedagogical application that will show you how to define and deploy a web application using Pharo / Seaside / Mongo and frameworks available in Pharo such as NeoJSON.

Our goal is that you will be able to reuse and adapt such an infrastructure to create your own web applications.


### Structure

In the first part called "Core Tutorial", you will develop and deploy, TinyBlog, an application and its administration using Pharo, the Seaside application web server framework as well as some other frameworks such as Voyage and Magritte. Deployment with Mongo DB is optional but it allows you to see that Voyage is an elegant facade to persist your data within Mongo.

In the second part and optional part, we will show you some optional aspects such as data export, use of Mustache or how to expose your application using a REST API.

Presented solutions are sometimes not the best.
This is done that way to offer you room for improvements.
Our goal is not to be exhaustive.
We present one way to develop TinyBlog nevertheless we invite the reader to read further references such as books or tutorials on Pharo to deepen his expertise and enhance his application.

Finally, to help you to get over possible errors and avoid getting stuck, the last chapter describes how to load the code described in each chapter.

![The TinyBlog application. % width=100&label=TinyBlogOnPharoCloudHere](figures/TinyBlogOnPharoCloud.png)


### Pharo Installation


In this tutorial, we suppose that you are using the Pharo MOOC image (it is currently a Pharo 8.0 image) in which many frameworks and web libraries have been loaded: Seaside (component-based web application server), Magritte (an automatic generation report system based on descriptions), Bootstrap (a library to visually tune web applications), Voyage (a framework to save your objects in document databases) and some others.

You can get the Pharo MOOC image using the Pharo Launcher ([http://pharo.org/download](http://pharo.org/download)).

### Naming Rules


In the following, we prefix all the class names `TB` (for TinyBlog).
You may:
- either choose another prefix (by example `TBM`) to be able to load the solution side by side to your own. This way you will be able to compare the two solutions,
- either choose the same prefix to fusion the proposed solutions in your code. The merge tool will help you see the differences and learn from the changes. This solution may be more complex if you implement your own extra functionalities.


### Resources


Pharo has many strong pedagogical resources and a super-friendly user community. Here is a list of resources:

- [http://books.pharo.org](http://books.pharo.org) proposes books around Pharo. Pharo by Example can help you to discover the language and its libraries. Entreprise Pharo: a Web Perspective presents other aspects useful for web development.
- [http://book.seaside.st](http://book.seaside.st) is one of the books on Seaside. It is currently under migration as an open-source book [https://github.com/SquareBracketAssociates/DynamicWebDevelopmentWithSeaside](https://github.com/SquareBracketAssociates/DynamicWebDevelopmentWithSeaside).
- [http://mooc.pharo.org](http://mooc.pharo.org) proposes an excellent Mooc with more than 90 videos explaining syntactical points as well as object programming key concepts.
- A discord channel where many Pharoers exchange information and help each other is accessible here: [http://www.pharo.org/community](http://www.pharo.org/community)

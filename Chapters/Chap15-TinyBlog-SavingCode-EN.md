## Save your code


When you save the Pharo image (Pharo Menu > Save menu entry), it contains all objects of the system as well as all classes.
This solution is useful but fragile.
We will show you how Pharoers save their code directly using Iceberg.
Iceberg is the Pharo code versioning tools (introduced in Pharo 7.0) that directly send code to well-known Web sites such as: github, bitbucket, or gitlab.

We suggest you read the chapter in the book "Managing Your Code with Iceberg" \(available at [http://books.pharo.org](http://books.pharo.org)\).

We list the key points here:

- Create an account on [http://www.github.com](http://www.github.com) or similar.
- Create a project on [http://www.github.com](http://www.github.com) or similar.
- Add a new project into Iceberg by choosing the option: clone from github.
- Create a folder `'src'` with the filelist or using the command line in the folder that you just cloned.
- Open your project and add your packages (Define a baseline to be able to reload your code -- check [https://github.com/pharo-open-documentation/pharo-wiki/blob/master/General/Baselines.md](https://github.com/pharo-open-documentation/pharo-wiki/blob/master/General/Baselines.md))
- Commit your code.
- Push your code on github.

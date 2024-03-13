# TinyBlog

TinyBlog is a tutorial (in English) for [Pharo](www.pharo.org).
It explains how to create a simple blog engine with Pharo's web stack (Seaside, Magritte, Voyage, Bootstrap, etc.).
Pay attention the book mentions to use Bootstrap4 but this is wrong since the code examples were not correctly updated.
 
[The latest PDF](https://github.com/SquareBracketAssociates/TinyBlog-EN/releases/download/latest/TinyBlog-EN.pdf) of this book.


Pay attention this book is getting old. 

Here are the different versions of the Bootstrap binding according to their author - torsten Bergman
We thank him for the support. 


## 1. Bootstrap v3.x framework

This was the original wrapper that I wrote "ages ago", the full source code still is available in repo:

```
           https://github.com/astares/Seaside-Bootstrap
```     

The class prefix back then was called "TBS" and there was an additional method prefix "tbs" used, so selectors where looking like #tbsSomething.

```
     Metacello new  
           configuration:'Bootstrap';  
           repository: 'github://astares/Seaside-Bootstrap:master/src';  
           version: #stable;  
           load
 ```

It can still be loaded and used that way also in the latest Pharo 11 - all 294 tests pass and are green.

=> This is the one that I know the MOCC originally used and I think as of today the code examples for TinyBlog use these "outdated" "tbsXXX" methods

## 2. Bootstrap v4.x framework

Later a new version 4.x of the Bootstrap CSS framework appeared in pubklic - it had some primary changes. So also for my new Pharo wrapper I did not try to stay compatible with the old wrapper - so I did a full new reimplementation that tried to stay as close as possible to the Bootstrap framework. Even when this meant breaking changes.

The source code for this is hosted in an own repo:

```
            https://github.com/astares/Seaside-Bootstrap4
```
    and the load instructions are:

```   
    Metacello new
        baseline:'Bootstrap4';
        repository: 'github://astares/Seaside-Bootstrap4:master/src';
        load
```

To signal that it is different (and also breaking compared to the old one)  I used a different class prefix "SBS" (instead of "TBS") and there was no additional method prefix anymore to also follow Pharo best practices. 

As of today all 499 tests pass and are green also in the latest Pharo 11.

Due to the BREAKING CHANGES between v3.x. and v4.x. version the GitHub page of the v4.x. version also mentions a few migration steps:
- the prefix is gone, so use container instead of tbsContainer, etc.
- use formButton or outlineButton instead of tbsButton buttons do not have beExtraSmall and beExtraSmallIf: styles anymore
- breadcrumb section is now called breadcrumb item

When I check the current MOOC PDFs in http://rmod-pharo-mooc.lille.inria.fr/MOOC/PharoMOOC/TinyBlog/ in chapter 5.2.
I think someone just changed the repo URL in the tutorial to point to the newer "Seaside-Bootstrap4" repo but did not take
the time to test and update the code examples (they are all still with the "tbsXXX" prefix from the 3.x version)
 
I think this MISMATCH IN THE TUTORIAL creates the trouble and confusion when people now LOAD THE BOOTSTRAP 4 (OR 5) BUT TRY
THE EXAMPLES WITH THE OLD "PREFIXED" METHODS OF BOOTSTRAP 3. 

This can only be fixed when someone updates the tutorial code examples (or rolls back to the old repo location in Chapter 5.2 to point to the old v3.x repo). I do not know if the videos from MOOC have similar trouble.
             
## 3. Bootstrap v5.x framework

Meanwhile there is already a new version 5.x of the Bootstrap CSS framework - and there is also a new repository for it

 ```
     https://github.com/astares/Seaside-Bootstrap5
```

This is the MOST RECENT VERSION OF THE WRAPPER - it continues to use the same class prefix "SBS" now and is compatible with the v4.x
version. The load instructions are


```
     Metacello new
         baseline:'Bootstrap5';
         repository: 'github://astares/Seaside-Bootstrap5:master/src';
         load
```
and all 510 tests are green in Pharo 11 as of today.

=> This is the latest, maintained code I can provide and recommend

 
## So my recommendation is:

a) short-term: tell your students to load the code from the v.3.x repo https://github.com/astares/Seaside-Bootstrap (as long as the tutorial has not been updated) as this will provide you with the original "tbsXXX" methods for the TinyBlog examples. This should work also in latest Pharo 11.

b) mid-term: someones should ROLLBACK the bootstrap project URL in the MOOC tutorial in chapter 5.2 from 
https://github.com/astares/Seaside-Bootstrap4 (v4.x) back to https://github.com/astares/Seaside-Bootstrap  (v3.x)
so it will not be a problem for people who just read the wrong PDF to follow the tutorial now
or better:

c) long-term: someones should UPDATE the MOOC tutorial to mention the v5.x project URL and adopt and test the TinyBlog examples to use this new v5.x version. Then this will nuke the problem in full and the latest Bootstrap code is used. 
This will solve it then once and for all.
 

For further questions feel free to contact me via chat in the Pharo Discord server (https://discord.gg/QewZMZa) easily.

Hope this helps! Слава Україні!

Regards and greetings from germany

Torsten (AKA "astares" on Pharo Discord)

Torsten (AKA "astares" on Pharo Discord)

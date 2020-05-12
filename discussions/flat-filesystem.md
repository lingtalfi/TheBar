Flat file system
===============
2020-05-12



An alternate file system.



Intro
--------

As I was looking for a way to store files uploaded by the users, I discovered what I call a "flat file system".
Weighting the pros and cons against the regular filesystem, I decided that it was a solution that better fitted my needs.

In this document, I'll explain what the flat file system is, and why I prefer it over the regular filesystem to store files
uploaded by users.



The flat file system
-----------

In a regular filesystem, every file has a unique path which serves as the identifier of the file.
This path has two parts: a directory and a filename.


Here is an example of what an arbitrary structure looks like with the regular filesystem:

```txt
- my_fruits/
----- yellow/
--------- banana.png
--------- lemon.png
----- green/
--------- apple.jpg
```



In the flat filesystem, every file has a unique identifier, which serves as the identifier of the file, and some metadata associated with it.
The metadata can contain anything and is stored in an external "store", which can be anything (a database, a file, etc...).


To emulate a regular filesystem, we need the following metadata:
- directory 
- filename

Here is how the same arbitrary structure would look like in the flat filesystem:

```txt
- id001
- id002
- id003
```

In the external store for this structure, we would have the following relationships:

```yaml
files:
    id001:
        filename: banana.png
        directory: my_fruits/yellow
    id002:
        filename: lemon.png
        directory: my_fruits/yellow
    id003:
        filename: apple.jpg
        directory: my_fruits/green
``` 



Pros and cons
---------


### Pros


The benefits of the flat file system are the following:

- it eliminates/simplifies the need for a sanitization phase that could potentially break the regular filesystem.
    This includes: 
    - checking the filename's length (you don't want the user to provide a filename that's too long), although it's still a good idea to do so
    - checking the directory's length (you don't want the user to provide a directory that's too long), although it's still a good idea to do so
- it eliminates the need for checking filename conflicts.
    This is a limitation of the regular file system which cannot have two files with the same path.
    With the flat filesystem, files can have the same directory/filename combination without conflicting with each other.
    Generally, the user wants to have files with separate directory/filename combinations, but he can always rename his files (i.e. let the user do the work). 
- we can store more metadata if necessary (file creation time, file last update time, file is public or private, author, etc...)



### Cons

The cons of using a flat filesystem are:

- we cannot use the regular visualization tools to see the intended file organization.
    This includes:
    - terminal commands, such as `ls`, `tree`
    - the OS default file viewer (i.e. Finder in Mac and Explorer in Windows)



My preference
----------

As to store uploaded files that come from the users, I prefer using a flat filesystem, because there are a lot of checking to do,
and getting rid of those checking is a big plus for me, as it makes the app conceptually simpler. 

As to visualizing the file organization, it seems quite trivial to create alternative tools to visualize the structure.
This includes terminal tools and/or web based tools (i.e. to visualize the structure in a browser).






Visualizing tools
-------

This section contains some visualizing tools we can use to see the structure of a flat filesystem. 

At the time of writing this document, the flat filesystem is just an idea, and I've not implemented a visualizing tool yet.




 







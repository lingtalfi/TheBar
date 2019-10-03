Php Secure file upload
=====================
2019-10-02





Summary
=========
- [Introduction](#introduction)
- [Hacking ourselves tutorial](#hacking-ourselves-tutorial)
- [My recommendations](#my-recommendations)
- [Sources](#sources)



Introduction
===========

In this tutorial, I'll talk about how I believe a web application should be secured as far as file upload is concerned. 


I'm not a security expert, but here is what I believe after reading a few documents.



First off, if you want to delegate the file management problems, you can use external services such as Amazon S3.

Now the rest of this document is about trying to handle secure file uploads ourselves (meaning using our own servers) using php.


There are basically two phases to a file upload attack:


- the attacker needs to upload a malicious file on the server
- the attacker needs to execute that file

Rather than discussing those two steps separately, let's just do a quick walkthrough that showcases those two steps into
one smooth linear approach.


Let's try to hack ourselves! 



Hacking ourselves tutorial
==============


Create a file named **phpinfo.php** and put the following content in it:


```php 
phpinfo();
```


If you open it in a browser. Since it has the php extension, your server will (depending on its configuration) execute the php code in it,
and you should see the output of the phpfinfo() function.


Now add the gif extension: **phpinfo.php.gif** and open the file in a browser.
Now your server might think it's a gif file and interpret it as such (depending on the server configuration), and so you'll see a failing gif image (since it's not a valid gif image).
In other words, the php code is not executed.


So this first point is very important: how your server serves the file.

Some people claim to have trick the server to think of the gif extension as a php file.
We shall evaluate that assessment, but in short, I don't believe it with nowadays' server configuration.


However, now create a **trigger.php** script with this content:

```php
<?php

include 'phpinfo.php.gif';
```

And then open **trigger.php** in a browser. Now the server executes a php file, and the include function (and its relative cousins
include_once, require, require_once) will execute the php code in the file.
As a result, you'll see the output of the phpinfo function again.

Now what a stupid idea to use include to parse a user's provided file, isn't it?
But still it's a good thing to know where the risks are.

Now instead of using include, let's just state the obvious: if we use file_get_contents or readfile instead, 
then php will just read the content and not execute it.

So for instance if your **trigger.php** now contains this:

```php
<?php

header("Content-type: image/gif");
echo file_get_contents('phpinfo.php.gif');
```

And you open **trigger.php** in a browser, you'll have the same result a opening **phpinfo.php.gif** directly in the browser.
And so a legit question is: why not let the user open the **phpinfo.php.gif** file in the browser directly, why using a php wrapper?

Well if you let the user open the **phpinfo.php.gif** files directly, it also probably means that you let them upload files directly in your web root,
because that's the only directory they have access to. 
Now I've read a lot that storing your files in the web root directory is a bad thing, but I'm the kind of guy that wants more proof, because
on the other hand using a php wrapper is just redundant code, and I won't do that unless I have a good reason to do so.

So what's the big deal with putting the user files in the web root directory?


People argue that if they can execute the file, it will execute the code in it.
Yes, but as we've just seen, they can't execute the code embedded in a gif file, because the server won't allow it.

Ok, but what if you allow storage of php files? I mean that's a valid thing to do (although risky).
Well if you allow ".php" upload of php files, and they execute them directly in the browser, then yes, that's a problem.

For this reason alone, I personally would won't put the user files in the root directory, because I don't want my application
being limited in what file types it allows for upload.

Now there is a second reason of putting them outside of the web root, which is even more strong.
Imagine you want to implement a permission system, where only the user who owns a (uploaded) file can access to it.
If your files are in the web root directory, anyone who can guess the file url has access to it by default (unless you configure your web server and/or your application otherwise).

Now it might be pretty hard to guess a file name out of the blue, but still.

Now in my case, I'm using a light application (https://github.com/lingtalfi/Light), which uses a so-called **front controller** (basically
all virtual traffic is redirected to /index.php, while existing files under the web root as served by the server as regular files).
So in my case, if the attacker guesses the url to an existing file under the web directory, he will have access to it.

So for me at least, it's conceptually simpler to just have the user files outside of the web root directory, so that this risk don't even exist at all.
Now it's just a matter of adding a simple permission layer on top of that.  

So that's two reasons for me to not put user files in the web root directory. Your mileage might vary.



Now for the sake of the discussion, let's try other people fears and try to bust the myth out of them, or acknowledge them as valid points if they indeed are.


Embed php code in exif data
-------------

It's apparently possible to embed php code in exif data, so that you end up with a file that's a valid gif file (or jpeg, or whatever...),
and which contains interpretable php code.

Now the question is: if we create such a file, would the server execute the php code?
As we've seen above, my guess is: nope it won't, because the server will interpret it as a gif file.

But let's try that nonetheless, to eliminate every doubts.

Create a gif file (time for opening photoshop again) named **malicious.gif**.
Open it in a browser, you should see the gif image, meaning it's a valid gif file.

Now to write exif data to a file, I personally I'm on mac, so I'll use exiftool (brew install exiftool),
and execute the following command:

```bash
exiftool -Comment='<?php phpinfo(); ?>' malicious.gif 
```

Check that your exif data is indeed embed in the file using this online tool: http://exif.regex.info/exif.cgi

Now let's test the php out of it. Add the php extension to it so that it temporarily becomes **malicious.gif.php**
and open it in the browser. You should see the output of the phpinfo function, and that's again because the server interpreted
the file as php, and some valid php code was found in it (in the form of exif comment).
So that php is valid.

Now revert the file name back to **malicious.gif** and open it in a browser, does it execute the php?
Nope, again because the server will see this as a gif file and will call its gif handler which does not execute php code.
However, you'll see the gif image, because it's a valid gif image.


So we start to understand that the whole game for the attacker is about tricking the server to execute the php code in files.



Before we go further, does this work for bash code too? Can the server execute a bash file out of the blue?

Let's find out.

Create the **date.sh** file with the following content in it:

```bash 
date
```

And open it in a browser.
In my case, the browser asks me to download it, which means: it's not interpreted as bash on the server.
So I guess in 2019 servers don't have or don't use shell code handlers by default.

So that's one less potential trouble. 


However, to understand the magnitude of trouble you're into if you allow php code executing: simply imagine
that the attacker can remove all your files.
Here is an interesting example I found on the internet:

Create a malicious file named **malicious.phtml** with the following content in it:

```php
<?php
echo shell_exec($_GET['cmd']);
```

And once uploaded, call it in a browser like this: /malicious.phtml?cmd=whoami.
Are you freaking out now?
Well, don't, just make sure no php code is executed ever on a file uploaded by an user.






Now back to myths busting with php code, how attackers trick the server to execute a file as php?


### idea 1: they don't, they hope for a developer mistake like using the include function

Well, you know what to do in this case: just don't use include ever.

    
### idea 2: the malicious .htaccess
    
Attackers can upload their own .htaccess file, like this one: https://github.com/wireghoul/htshells.

Oops, that's not a myth, that's a real problem.

The fix is quite simple: don't allow a user to upload a .htaccess file ever.


### Some other recommendations from owasp

https://www.owasp.org/index.php/Unrestricted_File_Upload

I don't necessarily implement all of the one below, but I thought it would be a good idea to keep it in mind: 


- Never accept a filename and its extension directly without having a whitelist filter.
- The application should perform filtering and content checking on any files which are uploaded to the server. Files should be thoroughly scanned and validated before being made available to other users. If in doubt, the file should be discarded.
- It is necessary to have a list of only permitted extensions on the web application. And, file extension can be selected from the list. For instance, it can be a "select case" syntax (in case of having VBScript) to choose the file extension in regards to the real file extension.
- Uploaded directory should not have any "execute" permission and all the script handlers should be removed from these directories.
- Restrict small size files as they can lead to denial of service attacks. So, the minimum size of files should be considered.
- Adding the "Content-Disposition: Attachment" and "X-Content-Type-Options: nosniff" headers to the response of static files will secure the website against Flash or PDF-based cross-site content-hijacking attacks.
        It is recommended that this practice be performed for all of the files that users need to download in all the modules that deal with a file download. 
        Although this method does not fully secure the website against attacks using Silverlight or similar objects, it can mitigate the risk of using Adobe Flash and PDF objects, especially when uploading PDF files is permitted. 
- Flash/PDF (crossdomain.xml) or Silverlight (clientaccesspolicy.xml) cross-domain policy files should be removed if they are not in use and there is no business requirement for Flash or Silverlight applications to communicate with the website.













My recommendations
==============


1. Upload all the user files outside the web root directory.
-------

In a light app, I'll suggest this:

```txt 
- app/
----- www/              the web root directory
----- user-data/        the root for all data uploaded by users
```



2. Implement a blacklist of forbidden filenames
---------

The file list should look like this:

- .htaccess
- .git
- crossdomain.xml
- clientaccesspolicy.xml
- ...? more depending on your use case


3. Use portable filenames only, and limit the length
-----------

I didn't mention it before, because it seemed obvious to me, but you should now allow weird characters in filenames.
You can use the [portable filename idea](https://github.com/lingtalfi/NotationFan/blob/master/portable-filename.md),
combined with a length checking (file with names longer that 64 or 128 chars for instance should be refused).


4. Implement a whitelist of extensions
---------

Depending on your application requirements, allow certain file extensions only, and make sure they are handled by the web server
as you expect.


4. Limit the max file size depending on your app requirements
--------------

Also, don't forget about some php directives that can help you:


Also, you can use MAX_FILE_SIZE in your forms to prevent legit user to upload too heavy files by inadvertence: https://www.php.net/manual/en/features.file-upload.post-method.php

- post_max_size
- upload_max_filesize






Sources
===========
- https://paragonie.com/blog/2015/10/how-securely-allow-users-upload-files
- https://websec.io/2012/09/05/A-Silent-Threat-PHP-in-EXIF.html
- https://www.owasp.org/index.php/Unrestricted_File_Upload
- https://www.electrictoolbox.com/disable-php-apache-htaccess/
- https://medium.com/@Aptive/local-file-inclusion-lfi-web-application-penetration-testing-cc9dc8dd3601
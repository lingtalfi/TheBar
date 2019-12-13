Ajax file upload
================
2019-10-17




Uploading file by ajax rather than using the traditional file input is cool, because it's more fancy,
the user can see the result of his file directly, the developer can add progression bar, it's more interactive.

However in this discussion I would like to discuss a couple of points I found worth mentioning.


First, as a background for this discussion, there is the [ajax file box](https://github.com/lingtalfi/Chloroform/blob/master/doc/api/Ling/Chloroform/Field/AjaxFileBoxField.md),
which comments explain the general structure/idea of how an ajax file upload should be implemented (or at least my
thoughts about how it should be implemented).


But this ajax file box discussion leaves the server part, so I wanted to fill in the gap.
For the server, as an example I've already implemented one helper tool here (as a [light](https://github.com/lingtalfi/Light) service):


- [Light_AjaxFileUploadManager](https://github.com/lingtalfi/Light_AjaxFileUploadManager)



And basically I just wanted to add one word: about the file name.

This even applies to regular static uploads (traditional upload in $_FILES).

It's an idea that I call symbolic file names.



Symbolic file names
-------------------

When you upload a file, it takes some space on your hard drive.
If space matters for you, then you should be careful how many files a user should be allowed to upload,
otherwise you may end up with a full disk on your server, which results in a buggy/quirky server with unpredictable 
behaviour, I learned this the hard way.


Indeed, imagine a user keeps uploading some files on your server, maybe because he changes his mind, or maybe just
to annoy you (being a malicious user), you might end up with some file weight problems.

So a first line of defense is usually to restrict the maximum file size allowed, but I would say that it's not enough,
because as I said the user can have multiple tries, change his mind and upload another file, and another, and another...

Well, as long as you keep deleting the old files, that's ok.
So it's all about deleting the old files.

And that's where the **symbolic file name** comes into play.

At a semantic level, you generally want to allow the user to upload only a fixed number of file.
Usually one file for things like an avatar, or multiple files if you them upload a photo gallery for instance.

The thing is that you know how many maximum files are allowed, and the idea of **symbolic file names** is that
the files have unique persistent names.

This has a couple of benefits:

- first in your mind you can always reference the file with the same name, as if it was an object with a persistent name.
        As a developer, it feels logical to have only one file name when you call it, because that's how you name it,
        meaning you have total control over the name and only you is able to choose it.  
- it resolves the aforementioned weight problem. You don't have to think about deleting the old file, because
        you just keep overwriting the same file again and again.
        
        
The drawbacks:

- if you use a non-secure upload system, it makes it easier for an attacker to know the location of the file.
        But I would argue that this should never happen, because your server uploads should be secure.
        A secure file upload is discussed here: https://github.com/lingtalfi/TheBar/blob/master/discussions/secure-file-upload.md.
        But the main idea to me is that you should store the files outside the web root directory, 
        and use a virtual server to serve them, which ensures that the potential code inside the files is never executed.
        You do that by reading the content of it and sending it to the browser (i.e. never include the content, which
        would execute it, but read it and display it). 
        
        
- there is another well-known drawback, but it has a fix. See the "2 steps validation process" section below for more details.        
         


So I hope this makes you aware of the file weight problem, in case you weren't already.





2 steps validation process
-----------------------
2019-10-21 

The 2 steps validation process is a workaround a well-known problem that arises when one implements an ajax file upload system using **symbolic file names** (see the above section for more details).
When you use an ajax form with symbolic file names, each time you upload a file, the file gets overwritten.

So for instance, if the user fills a profile form, and the symbolic file name for the avatar picture is **images/avatar.png**, 
then every time the user uploads a new avatar, the user file (ending with **images/avatar.png**) will be overwritten.

This is a problem, because the user didn't get a chance to submit the form, and yet the avatar is already written to the file system.
This means the user can't cancel and gets his/her old avatar back. That's the problem. 

Therefore, I introduce a simple fix called **2 steps validation process**.

The idea is the following:

- when the user uploads an avatar which symbolic file name is **images/avatar.png**, then a file is first created, which name is **images/avatar.2svp.png**
    The 2svp extension indicates that this file is temporary and is awaiting for a validation phase.
    Note that the user can changes her mind and upload a new avatar as many times as she wants, the same **images/avatar.2svp.png** will be overwritten.
- then in the end, the user posts the form. We basically simply remove the **2svp** extension from the file.
        Note: this might also include other operations such as updating a database, depending on the application.
        






Related
==========
- [Secure file upload discussion](https://github.com/lingtalfi/TheBar/blob/master/discussions/secure-file-upload.md): a discussion about secure file upload in general
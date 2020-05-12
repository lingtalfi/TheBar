The file manager protocol
==============
2020-04-06




This describes the communication between a js client and a server which allow the user to manage his files.

By managing, I mean:

- the user has a dedicated space on the server where he can finds his files
- the user can upload new files
- the user can delete his files
- the user can edit the information of his files (he can change the name, and meta information such as tags)
- the user can decide whether the file is private or public (private means the file can only be accessed by him)
- in the case of an image, the user can also update the file with an image editor (cropping, filter effects, ...whatever is supported by the client) 



Now the image editor capabilities are defined by the client alone and not part of this protocol.


In this document, **acp** means [ajax communication protocol](https://github.com/lingtalfi/AjaxCommunicationProtocol), which is a json based protocol.

All actions, unless otherwise specified communicate via **acp**.



How do the client and server communicate?
=========


The available operations are:


- add: to add a file on the server
- delete: to delete a file owned by the user 
- update: to update the information of a file owned by the user  
- get_partial_size: to get the current size of a partially uploaded file
- reset: to reset the virtual server (in case the server exposes a virtual server)


In addition to that, the **urls** returned by the server on the **add** action must contain additional
meta information described in the **file manager urls** section of this document, and the client must know how to treat this meta information.    





reset 
-------
If your server uses a virtual file system such as [this one](https://github.com/lingtalfi/Light_UserData/blob/master/doc/pages/conception-notes.md),
then the client must send the reset action to the server every time the page is reloaded, and/or when the user resets the gui (i.e. when he clicks the reset button of the form for instance).

There is no particular payload for this operation.

The server must then reset the virtual file system when it receives this command, and respond with any success alcp response.  



add 
-------

The add operation accepts two modes: 
- regular upload: the whole file is uploaded at once on the server 
- chunk upload: the file is sliced in chunks and every chunk are uploaded. It's possible to interrupt the upload and resume it later if the
    server configuration (and the js client) allow it. If you have no particular need, this is the recommended method, as it will
    handle more cases (especially if your application allows your users to upload big files). See the "keepOriginalUrl" section in this document for more details
    
    
The js client decides which mode to use with the **useChunks** property.    


The client provides the file (js File object) and all the meta-information it has if the user has provided them already (such as tags, or 
the file name that the user has changed).


The data is passed via the POST method, multipart/form-data style (https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST).


 
- configId: the process instructions id. The server will know how to handle the upload (once finished) with this id. This ensures that the 
    server has total control over every upload, since upload is a common vector for malicious attacks.
   
- useChunks: 0|1, whether to use the **chunk upload** mode or the **regular upload** mode.
- file: the js File object to upload

- ?filename: the name of the file (otherwise the server might take a guess based on the uploaded file).
- ?directory: the relative path of the directory which contains the file. 
- ?tags: an array of tags to attach to the file
- ?is_private: 0|1 (defaults to 0), whether the file should be private or public
- ?keep_original: 0|1 (see the keep_originalUrl section for more details).
 

In **chunk upload** mode, the following properties must be added:

- start: the byte at which the slice (of the whole uploaded file) starts
- end: the byte at which the slice ends (byte excluded)
- last_chunk: 0|1, whether this slice is the last one of the file


If the user interrupts the uploading, then, if the server (and js client) allow it, it's possible to resume the upload later.
To do so, the client first acquire the current size of the partially uploaded file by using the **get_partial_size** operation (described elsewhere 
in this document). Then he just executes the **add** operation using the **chunk upload** mode with the start value being the size of the partially uploaded file. 


Note that all information passed via the user might be overridden by the server, due to security reasons.
In particular, the directory is typically often fixed by the server.
The file object can also be updated by the server (for instance if the server reduces an image so that it fits into a 200x200 square).



The server's response uses the [ajax communication protocol](https://github.com/lingtalfi/AjaxCommunicationProtocol).
In case of a success, the json array contains the following properties.

Remember that the server can change any information provided by the user (hence we provide the data).

   
- is_fully_uploaded: 0|1. 0 means that this was just a chunk that was uploaded.
    1 means the the full file has been uploaded (in a case of the chunk upload strategy, this means that all the chunks have been uploaded,
    and the file has been rebuilt successfully on the server).
    If the value is 0, all the other properties are not returned.  
- url: the url to the uploaded file  
- filename: the name of the uploaded file  
- directory: the directory path (relative to the user's directory) which contains the uploaded file  
- tags: the array of tags attached to the uploaded file  
- is_private: whether the file is private or public
- original_url: string=null. The url to the original file if there is one, or null if there isn't. 


Note that in the case of an image, the server might have altered the uploaded image to fit certain dimensions requirements (i.e. the 
server might have cropped the uploaded image a bit). Therefore in this case the js client shall request the new image (via the given
url) to update its visual representation in the gui.  




delete
----------

With this action, the client ask for the server to remove a resource from the server.


The js client sends the following payload via post:

- url: the url of the file to remove


The server will respond with the regular **acp** response.








update
-------


This allows the client to update a resource on the server.

The client must send the same parameters as with the **add** action, but with the following extra parameters:

- url: string, the url of the resource to update


The server will respond with a regular **acp** response.


get_partial_size
----------

This operation returns the size in bytes of a partially uploaded file.
Not all server will allow this because it involves keeping track of those partially uploaded file.

The js client must send the following properties:

- filename: the name of the file
- ...? maybe other properties will be added in the future


The server responds with an **acp** response. If the file exists:

- size: the size in bytes of the partially uploaded file 


  



keepOriginalUrl 
===========

This is an extra feature that allows the user to recall an image in its first state (i.e. when it was first uploaded),
allowing him to test different cropping without having to re-upload the original image.


When working with images, cropping is an interesting feature.
However if the user keeps cropping the same file again and again, the image size gets smaller and smaller until there is nothing left to crop.

Therefore in some cases it's a good idea to provide the original file rather than the cropped version, so that the user can change his mind
and re-crop the image as many times as he wants without having to worry about the file dimensions to be reduced every time.

So the idea is simple: when the user adds a new file, the server memorizes that file.
Then with the gui, the user has the option to access that original image again. 


Implementation ideas
-----


The original file is called **original file**. There is only one **original file** max per url.
All the variations of the user are called a **variation**. Although the user can test as many variations as he wants, there is always
one variation per url at the time.

When the user wants to upload a file and keep the original, he sends the keep_original=1 flag to the server along with the payload
for an add operation.

When receiving that flag, the server stores the **original file** in a safe place.
From now on every time a request is made for that url, the server will add the **original_url** property via the response headers and/or the response,
depending on what's more practical.

The js client receives the **original_url** value and stores it in its file representation, and handles the recall to that url when necessary (i.e. when the
user asks for it via a gui button for instance). 
 







File manager urls
=========
2020-04-13

When a successful **add** action is performed, the server returns an **url**. Along with this url, the server provides information
necessary to make the file manager protocol work correctly.

This meta information contains the following properties:

- original_url: string, the url to the original url if any, see the **keepOriginalUrl** section for more details



The information is delivered via the [panda headers protocol](https://github.com/lingtalfi/TheBar/blob/master/discussions/panda-headers-protocol.md), and headers are
prefixed with the **fmp_** string.
Therefore, the panda headers will contain the following:

- fmp_original_url: (value for the original_url property)
















Old crappyThe fileEditor protocol addition 
------------
2020-01-28 -> 2020-02-21

The fileEditor protocol addition is an extension of the ajax file upload protocol.
The goal is to provide the user with a more powerful file management experience.


The backend service is willing to handle the following extra-parameters (note that all of them can be overwritten by the server):

- extension: mandatory, string = fileEditor.
- action: optional, string(add|remove|update)=add.

    This defines the type of action to execute. The two choices are **add**, **remove** and **update**.
    With the **add** action, the intent is to add a new file to the server.
    The **add** action might trigger an error if the file we are trying to create already exists in the server (i.e. name conflict),
    depending on the server configuration.
    
    The **remove** action will delete an existing file. An error will be thrown if the user tries 
    to remove a non-existing file or a file she has not permission on.
    
    The **update** action intent is to update information about the file, and/or the file itself.
    Again, same as with the **add** action, if the updated file location already exists, the operation might be rejected 
    by the server, depending on the server configuration.
    
    Depending on the action type, the parameters to send to the server will differ, and so might the server's response.
    
    
     
- Params for the **add** action:             
    - filename: mandatory, string.
    
        The file path (including file extension) chosen by the user.
        How it's used by the server depends on the server configuration: it might be just a filename which the server
        would put in a predefined directory, or it could be a portion of path if the server configuration allows the
        creation of subdirectories.        
        
        The server might even overwrite totally or partially the filename in order to provide
        a better service (for instance the server could decide to choose the file extension).
        
    - is_private: optional, string=0|1.
    
        Indicates whether the file should be considered as private (only the user should be able to see it) 
        or public (anybody can see it).
        0 means public, 1 means private.
        The server might not understand that parameter, check your server before using that parameter.
        
    - tags: optional, array=[].
    
        An array of tags to attach to the file.
        It's an array of id => label,
        where id is the identifier of the tag.
        The server might not understand that parameter, check your server before using that parameter.
        
        
- Params for the **remove** action:
    - url: mandatory, string. 
    
    The url of the file to remove.     
    
- Params for the **update** action:
    Same params as the params for the **add action**, but with one extra property:
    - url: mandatory, string. The url of the file to update     
         



Response for the **add** action: same as the standard response.
Response for the **remove** action: same as the standard response, but the url parameter is not sent back.
Response for the **update** action: same as the standard response.




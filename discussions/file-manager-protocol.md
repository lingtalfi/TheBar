The file manager protocol
==============
2020-04-06 -> 2020-05-22




This describes the communication between a js client and a server which allow the user to manage his files.

By managing, I mean:

- the user has a dedicated space on the server where he can find his files
- the user can upload new files
- the user can delete his files
- the user can edit the information of his files (he can change the name, and meta information such as tags)
- the user can decide whether the file is private or public (private means the file can only be accessed by him)
- in the case of an image, the user can also update the file with an image editor (cropping, filter effects, ...whatever is supported by the client) 



Now the image editor capabilities are defined by the client alone and are not part of this protocol.


In this document, **acp** means [ajax communication protocol](https://github.com/lingtalfi/AjaxCommunicationProtocol), which is a json based protocol.

All actions, unless otherwise specified communicate via **acp**.



How do the client and server communicate?
=========
2020-04-06 -> 2020-05-22

The available operations, exposed by the server, are:


- **add**: to add a file on the server
- **delete**: to delete a file owned by the user 
- **update**: to update the information of a file owned by the user  
- **get_partial_size**: get the current size of a partially uploaded file, used to resume a paused chunk upload (only relevant if you use a chunk uploading system such as the [simple chunk upload protocol](https://github.com/lingtalfi/TheBar/blob/master/discussions/simple-chunk-upload-protocol.md))
- **reset**: to reset the virtual server (only relevant if your server uses a virtual file system such as the [TemporaryVirtualFileSystem](https://github.com/lingtalfi/TemporaryVirtualFileSystem/blob/master/doc/pages/conception-notes.md))


In addition to that, the server provides a way for the client to access a file by its **url** (more info below in this document).    




To trigger an operation, the client must send a POST request to the server, 
multipart/form-data style (https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST),
with a parameter **action** which value is the name of the operation to execute.

Each operation has also some specific parameters described below.

The server responds with a standard **acp** response.



add 
-------
2020-04-06 -> 2020-05-22

The **add** operation accepts two modes: 

- **regular upload**: to upload the whole file at once on the server.

    This is not recommended, since in the case of big files, the server could reject the upload.
    With php, you might need to configure directives such as **post_max_size** and **upload_max_filesize**
      
- **chunk upload**: to send the file in small chunks.

    This is recommended because you don't need to worry about the server rejecting your file, no matter which size it is. 
 
    With this mode, it's also possible to interrupt the upload at any moment and resume it later if the
    server configuration (and the js client) allow it (see the **get_partial_size** operation for more info). 
    

The extra parameters for the **add** action are the following:

    
        
- **useChunks**: 0|1, mandatory.
    Tells the server whether to use the **chunk upload** mode, or the **regular upload** mode.
        
- **file**: binary data, mandatory.
    The binary data representing the file (or chunk if you're using the **chunk upload** mode) to upload.


If you use the **chunk upload** mode, the following parameters are required:

- **start**: int, the byte at which the slice (of the whole uploaded file) starts
- **end**: int, the byte at which the slice ends (byte excluded)
- **last_chunk**: 0|1, whether this slice is the last one of the file



In addition to that, the client can send any other parameter it wants to the server.



### Standard set

As an example, in my own implementation I use the following set of extra-parameters, which I call **standard set**:

- **configId**: string, mandatory.

    The process instructions id. The server will know how to handle the upload (once finished) with this id. 
    This ensures that the server has total control over every upload, since upload is a common vector for malicious attacks.
    
- **filename**: string, optional.
    The name of the file.
    This represents the wish of the client, but the server can overwrite this value if it wants to.
        
- **directory**: string, optional.
    The relative path of the directory which contains the file.
    This represents the wish of the client, but the server can overwrite this value if it wants to.
    
     
- **tags**: array, optional.
    An array of tags to attach to the file.
    
    
- **is_private**: 0|1=0, optional.
    Whether the file should be private or public.
    
- **keep_original**: 0|1=0.
    Tell the server to keep a copy of the file (see the **keepOriginalUrl** section for more details).
 


Note that for security reasons, the server can override any parameter passed by the client.


### Server's response

In case of a successful response, the server returns the following properties via the standard **acp**:
   
- **is_fully_uploaded**: 0|1

    0 means that this was just a chunk that was uploaded.
    1 means the full file has been uploaded (in a case of the chunk upload strategy, this means that all the chunks have been uploaded,
    and the file has been rebuilt successfully on the server).
    
    If the value is 0, the other properties are **NOT** returned.  
    
- **url**: string, the url to the uploaded file  

If you are using the **standard set** of parameters (described in the section above), then the server will return
the extra-following parameters:

- **filename**: string, the name of the uploaded file  
- **directory**: string, the directory path (relative to the user's directory) which contains the uploaded file  
- **tags**: array, the array of tags attached to the uploaded file  
- **is_private**: 0|1, whether the file is private or public
- **original_url**: string=null. The url to the original file if there is one, or null if there isn't.

 

Note that in the case of an image, the server might have altered the uploaded image to fit certain dimensions requirements (i.e. the 
server might have cropped the uploaded image a bit). Therefore, in this case the js client shall request the new image (via the given
url) to update its visual representation in the gui.  




delete
----------
2020-04-06 -> 2020-05-22

With this action, the client ask for the server to remove a resource from the server.


The js client sends the following payload:

- **url**: string, the url of the file to remove


The server will respond with a regular **acp** response with not particular properties in it.







update
-------
2020-04-06 -> 2020-05-22


This allows the client to update a resource on the server.

The client sends the same parameters as with the **add** action, except that it can selectively choose which ones to update.
So for instance it doesn't have to resend the file binary data if that hasn't changed.

So this means that the **file** parameter is optional.


Also, the following extra parameters must be provided:

- **url**: string, the url of the resource to update


The server will respond with the same **acp** response as with the **add** action.

Remember that the server can override the client's value, and so the client should parse the server's response, and
update the gui accordingly in order to keep synchronization between the state of the file in the server and in the gui.




get_partial_size
----------
2020-04-06 -> 2020-05-22

If you use the **chunk upload** mode, then it's possible to let the user interrupt the uploading at any moment, then resume it later.

To do so, the client must first acquire the current size of the partially uploaded file by using the **get_partial_size** operation.

Then to resume the upload the client calls the **add** operation with the **chunk upload** mode again, but this time with the **start** parameter value
being the size of the partially uploaded file (the size returned by the **get_partial_size** operation).


The js client must send the following parameters:

- **url**: the url of the file from which we want to get the partial size


The server responds with an **acp** response. If the file exists:

- **size**: the size in bytes of the partially uploaded file 








reset 
-------
2020-04-06 -> 2020-05-22

This operation is only relevant if the server uses a virtual file system.
 
The client must call the **reset** operation every time the page refreshes, and every time when the user resets the gui (i.e. when he clicks the reset button of the form for instance).


There is no particular extra parameter associated with this operation.

The server responds with a standard **acp** response with no particular properties.  









  



keepOriginalUrl 
===========
2020-04-06 -> 2020-05-22

This is an extra feature that allows the user to recall an image in its first state (i.e. when it was first uploaded),
allowing him to test different crops without having to re-upload the original image every time (to avoid the image getting smaller and smaller
on every crop).

So the idea is simple: when the user adds a new file, the server memorizes that file.
Then with the help of the gui, the user has the option to access that original image again while editing the image. 

There can be only one **original file** max per url.

The client tells the server to keep the current image as an **original file** using the **keep_original** flag with a value of 1 in an **add** operation.

When receiving that flag, the server stores the **original file** in a safe place.

From now on every time the client accesses a file by its url, the server will add the **original_url** property as a meta-information 
via the response headers.

The js client receives the **original_url** value and can then show it back to the user when necessary (i.e. when the
user asks for it via a gui button for instance). 
 





Accessing a file by its url
============
2020-04-06 -> 2020-05-22



With the **file-manager-protocol**, when the client accesses a file by its url, it can provide additional flags to do different things.


The client sends the flags via **GET**.

The available flags are:


- **m**: string (0|1) = 0. Whether to add meta to the returned http response.

    If true, the meta will be added via the [panda headers protocol](https://github.com/lingtalfi/TheBar/blob/master/discussions/panda-headers-protocol.md).
    The actual list of meta depends on the server.
    
    It could for instance return a parameter named **original_url** (see the **keepOriginalUrl** section in this document for more info).
    
    If you're using the **standard set** of parameters, it could for instance return those parameters in the meta: is_private, directory, filename, tags, etc...
    
    
- **o**: string (0|1) = 0. This is only relevant if you are using the concept of original file.
    Whether to return the file targeted by the url (by default), or the original file associated with it (if any). 
    See the **keepOriginalUrl** section in this document for more details.
    
- **v**: string (0|1) = 0. This is only relevant if you are using a virtual file server.

    Defines whether the file comes from the virtual server or the real server.
    By default, the real server serves the file.
   


    

 












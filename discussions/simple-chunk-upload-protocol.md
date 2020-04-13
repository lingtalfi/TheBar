The simple chunk upload protocol 
================
2020-04-13



The simple chunk upload protocol (scup) defines the communication between a client and a server when the client wants to upload
a file in slices (aka chunks) to the server.


The need for this happens for instance when you want to upload a 1G file, and your server can't handle it.
So instead the client will slice the big file into chunks (which size is customizable) of 1M each (for instance), and send them to the server one by one,
thus ensuring that each 1M sent will not break the server.

With this technique, we can upload files of any size. However the server needs to re-assemble the chunks together when the last chunk is sent.


So this protocol defines the role of the client and the server in this type of communication.



The client
------------

The client sends the chunks one by one, in order (very important), with the following payload sent via POST (multipart/form-data):

- file: Blob, the js Blob representing the chunk to send
- ?filename: string, a suggested filename. This can be overridden by the server.
            This helps the server defining where to put the file.
            In order to avoid collision, the client must make sure that two different file uploads have two different filenames.
- start: int, the byte number at which this chunk starts.
        The first chunk always starts at 0.
- end: int, the byte number at which this chunk ends.
        The last chunk's end is always the file size in bytes.        
- last_chunk: 0|1. Whether this chunk is the last one.


For instance, if your file weights 100 bytes and you send chunks of 20 bytes, the client has to send the 5 following chunks (the numbers represent the **start** and **end** values to send):

- 0-20
- 21-40
- 41-60
- 61-80
- 81-100






The server
-----------


The server role is to handle the client's requests and re-assemble all the chunks in one file on the file system.

Any implementation will do.

Below is my own implementation idea.

It uses the concept of **partials**.
A **partial** is a file partially uploaded.

Its the uploaded file path, but with the extra **.part** extension in the end.



```php



$start = (int)$_POST['start'];
$end = (int)$_POST['end'];
$isLastChunk = (bool)($_POST['last_chunk']);

$dst = '/my_app/tmp/' . $fileName; // your application business rules here...
$dstPartial = $dst . ".part";


/**
 * If the user doesn't ask for a resume operation, then
 * we need to remove the old partial before starting the upload operation.
 * This use case happens when the file starts to upload the file but then abort,
 * but then upload again.
 * If we don't remove this partial, the partial keeps growing every time the user upload new chunks,
 * this ends up with an inconsistent file that IS NOT the same as the original file that the user wanted to upload.
 *
 */
if (0 === $start && file_exists($dstPartial)) {
    unlink($dstPartial);
}


if (false === file_exists($dst)) { // optional

    if (false === file_exists($dstPartial)) {
        FileSystemTool::mkfile($dstPartial, "");
    }
    file_put_contents($dstPartial, file_get_contents($src), FILE_APPEND);


    if (true === $isLastChunk) {
        FileSystemTool::copyFile($dstPartial, $dst);
        unlink($dstPartial);

        // here we're done, send a response back to the client if you need to...
    }

} else {
    $errorMessage = "The file $fileName already exists";
}
```








The resumable upload 
============ 


This idea extends the **simple chunk upload protocol**.

The idea is that the user can abort an upload at anytime and resume it later.

In order for this to work, all the js client needs to know is where to start.
 
Assuming that you're using my php server implementation, it turns out that this number
is exactly the size in bytes of the current partial (if any).


I didn't create a protocol yet for that, but basically the idea is the following.

The client first sends a request with the filename (or a fileId of sort) to get that partial size.

The server checks that a partial exists for the given filename (or fileId) and responds with the partial size in bytes (for instance 451120).

Now the js client can start the chunk upload, with start=451120. 

The server part is already handled (if you're using my server implementation idea).




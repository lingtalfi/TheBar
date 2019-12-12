Full ajax form
==================
2019-12-12



It's probably just me, but so far I used the [ajax-file-upload technique](https://github.com/lingtalfi/TheBar/blob/master/discussions/ajax-file-upload.md) when my forms had some input files in them,
to make the input being triggered as soon as the user inputs the file.

This led to an incredibly complicated design where basically the form treatment is split in two parts: one for the static form,
and one for the file sent via ajax as the user provides the file.

I've been doing this for years now, kind of accepting this complexity without even questioning it, until today, where I realized that
in fact all I need was a simple file preview, which can be done via javascript without ajax (using FileReader).

Duh (how stupid do I feel?)!


But what happen when the user posts the form and there are some errors in it? How do you handle the consistency of the
input file? 
That's another question that I solved (with complexity) with my previous technique, but which I will solve again today,
this time in a more elegant manner.



Ok enough suspense, here is my new form technique for hopefully the rest of my web dev life (I'm tired to change technique).


The trick is very simple: 

- first you display your static form as you would normally do (I use [Chloroform](https://github.com/lingtalfi/Chloroform))
- then you add some js helper (let's call it **full-ajax-form-handler.js**, or fafh in this document)
- the js helper (fafh) will do the following:
    - first: when the user provides a file, it displays a preview without making an ajax request: we just want to give the user
            a preview of what the file will be. Note: we can already think of cropper.js scripts in addition to the default preview functionality.
    - then it takes over the submit event, so that when the user submits the form, the whole data is sent via ajax to a given controller
    - the controller responds with a protocol yet to define, which basically either describes the errors of the form, or returns a success message 
            indicating that the form has been successfully treated server side (database update for instance).
            The fafh tool will inject the form errors (and/or success message) dynamically in the form.
            
            

Let's stop right there.
With just those features I can see two tremendous benefits over my older technique:

- there is just one controller to handle the form (i.e. vs two different processes with my older technique)
- because the request is posted via ajax:
    - the input file is persistent (or should I say the page is never refreshed so it's still there)
    - plus we can add a please wait overlay easily, which makes the gui feel more professional
    
    
In addition to that, we could have some other kinds of js helpers to validate the form controls dynamically, creating
error messages on the fly, and removing them as the error is fixed in real time, but that's outside the scope
of this discussion for now.


So, I've not implemented the new technique yet, so let's get to work...     
                         





 
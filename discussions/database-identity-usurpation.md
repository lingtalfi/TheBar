Database identity usurpation 
===============
2020-02-28


Context:
 
A web app that has users, and those users have at their disposal some actions that allow them to change
the state of the database (perhaps updating their profile, or deleting some items that they own via the database).


Problem:

Perhaps the biggest problem in such a context is that maybe the user Alice can alter the state of a row owned by user Bernard.

Typically, if you're using some kind of auto-admin generator, and that generator creates some impersonal services that allow
anybody to edit/delete/update records, based on the table name and the id of the record.

The problem with those impersonal tool is that it drops the concept of "user owning a record", and so if used with bad intentions,
those services become the malicious/dumb users target #1, as they could simply edit/delete/update records of any user, as long
as they have the row id.


Yet using an auto-admin generator is very useful and legit, so how do we combat this problem?


Note: it's assumed that a personalized service, as opposed to impersonal, is one that uses the user id directly from the connected user (i.e. stored in the php session for instance). 

Note2: depending on your application, you might also want to protect the "read" operation, as to protect some confidential information about an user. 




A possible defense mechanism in light
------------

As usual, there are probably an infinite number of ways to prevent that.
In this discussion below though, I'll expose a defense mechanism used in the [Light](https://github.com/lingtalfi/Light) environment, as this is the framework
I'm working on at the moment.

It might serve as an example for other implementations in other frameworks.

I'll confess that I've not implemented the solution yet, so what follows is more of some conception notes rather than a definite implementation.


So in Light we have this [Light_Database plugin](https://github.com/lingtalfi/Light_Database) which is basically used every time a database interaction is made (at least in the scope of the plugins I've personally developed).

And so this plugin sees every sql request.


This plugins provides (amongst other methods) the following standard methods that alter/read the state of the database:

- insert
- replace
- update
- delete
- fetch
- fetchAll

My idea is that IF "all impersonal actions that alter the database always" use any of those methods, we can simply use those methods and hook some kind of third party service that would
check a few things:

- whether the table is a candidate for "user owning" (i.e. some tables are just impersonal anyway, they don't belong to anybody but the application system)
- if so, whether the current user is actually owning the record he/she is trying to alter


So that sounds like it would take care of the problem, but the big question is: are "all impersonal actions that alter the database" using only the four aforementioned methods?
If not yet, well they have to (at least if you want to implement this solution).


So yes basically the part we need to be careful with this solution is make sure that ""all impersonal actions that alter the database" are indeed triggering the hook that checks the user identity, otherwise there is no point.  







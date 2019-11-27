Php session security
==============
2019-11-26



Sources:
- https://www.freecodecamp.org/news/session-hijacking-and-how-to-stop-it-711e3683d1ac/
- https://www.thesslstore.com/blog/subnetting-and-masks/



Practical implementation
----------


First, make sure your website is only accessible via https.
Use certbot for online websites, and this [ssl tutorial](https://github.com/lingtalfi/server-notes/blob/master/doc/apache-ssl-on-local-machine-mac.md) for your localhost machine (if you're on mac).
See my [server notes](https://github.com/lingtalfi/server-notes) for more details/tutorials.

Also don't forget to redirect all non-https traffic to https (as done in my tutorial on apache ssl local).




Then, add this the beginning of your php script

```php
ini_set( 'session.cookie_httponly', 1 ); // make sure the cookie is not accessible via js (document.cookie)
ini_set('session.use_only_cookies',1); 
ini_set('session.cookie_secure', 1); // the cookie is sent only via https (so it's value is encrypted and not accessible) 
```

Now the attacker should have no easy way to impersonate your session id, unless he has access to your physical machine
in which case you're dead anyway.


All of this obviously assumes that your app is immune to xss (i.e. htmlspecialchars).




Also, when the user authenticates, regenerate the session, I'm not sure it's very useful since we have already https in place,
but it doesn't hurt.

```php
session_regenerate_id();
```








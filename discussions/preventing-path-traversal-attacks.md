Preventing path traversal attacks
====================
2019-10-15


Until today, I've always ignored string encoding related problems, as I found them difficult to understand.

However now my application will be using an identifier provided by the client, and the identifier
will be translated to a portion of a configuration file path, and so I absolutely don't want the client
to be able to go up a directory by using the ../ notation (which is know as the path traversal attack: https://www.owasp.org/index.php/Path_Traversal). 

So I prevented that by doing this:

```php 
$id = "some data provided by the client";
$id = str_replace('..', '', $id); // prevent user from going up a directory
$file = $this->confDir . "/" . $id . ".byml";
```


But is it enough? or is it hackable?

That's what we are going to find out in this document.






As a teaser to this discussion, in php it's possible to represent the same string using multiple representations.

For instance test this code:

```php
a("\xC3\xA1" === "á"); // true if the file is encoded with utf-8 C form, which is my case
a("\x61\xCC\x81" === "á"); // true if the file is encoded with utf-8 D form, which is not my case
a("\xE1" === "á"); // true if the file is encoded with ISO-8859-1, false in my case (encoding everything with utf8)
```

In my case, the first line yields true, proving my point.
Your mileage might vary, it all depends on which encoding your file (holding the code) was written with.
Source: https://www.php.net/manual/en/language.types.string.php#language.types.string.details


So that's it for the teaser, now let's start digging more.




Data provenance
=============

Sources: https://www.cgisecurity.com/lib/URLEmbeddedAttacks.html


Where does the user data come from?

- $_GET
- $_POST
- file uploads
- html forms


Php special chars interpretation
------------

Source: https://www.php.net/manual/en/language.types.string.php

In php, if the string is enclosed in double quotes, we can write the dot traversal string in different ways:

- regular chars:        ../
- hexadecimal:          \x2e\x2e\x2f
- utf8:                 \u{002e}\u{002e}\u{002f}


So for instance, if we have a file named **/my/path/to/test.txt**, then we can do this:

```php 
$d = "/my/path/to/otherdir";
$f1 = "../";
$f2 = "\x2e\x2e\x2f";
$f3 = "\u{002e}\u{002e}\u{002f}";

a(file_exists($d . $f1 . "text.txt")); // true
a(file_exists($d . $f2 . "text.txt")); // true
a(file_exists($d . $f3 . "text.txt")); // true

```

As we can see, those strings are interchangeable as far as the dot traversal attack is concerned.

And so a legit question is: can an attacker pass those strings somehow? and if so how should we protect against them? 

We will answer this question later, but for now let's continue on some other topic.




Testing the url, the $_GET
----------------
 
To test my web application, I will just call my web application using a get parameter, such as:

```txt
http://myapp?t=abc 
```

In this case: t=abc is the interesting bit.
If I observe the $_GET array, I get:

```html
array(1) {
  ["t"] => string(3) "abc"
}

```


Now let's try to hack ourselves.
Let's say I'm a hacker and I want to call "../test", to go up a directory and call a potentially existing test.byml file.

So if I type this directly in the browser (firefox in my case):

```txt
http://myapp?t=../test 
```
 
 
I get the expected $_GET array:

```html 
array(1) {
  ["t"] => string(7) "../test"
}

```

But my sanitizing function so far should be able to detect and remove those two dots.


Let's continue with the attacks. 

Let's gather more information about our target: $_GET.

From the php manual: 

```txt 
The GET variables are passed through urldecode(). 
```

Source: https://www.php.net/manual/en/reserved.variables.get.php

So can we hack urldecode?


Well, urldecode apparently uses url encoding, which is also known as percent encoding.

Source: https://en.wikipedia.org/wiki/Percent-encoding

So the dot traversal string "../" can be written like this: "%2e%2e%2f".


Let's try this:

```txt 
http://myapp?t=%2e%2e%2ftest
```


In php, the $_GET array now looks like this:

```html 
array(1) {
  ["t"] => string(7) "../test"
}

```


Well, it's the same as before, so the sanitizing function will still work.


Let's continue.
The percent char itself can be encoded again. 
This is known as the double encoding attack.

Source: https://www.owasp.org/index.php/Double_Encoding


So the encoded version of "%" being "%25", let's try this:


```txt 
http://myapp?t=%252e%252e%252ftest
```


Now our $_GET array looks like this:

```php
array(1) {
  ["t"] => string(13) "%2e%2e%2ftest"
}
```

It's different, but it's garbage and harmless in terms of directory traversal, so no worries here either.


So as far as url encoding, I'm not afraid of those.



Utf8-encoding
----------------

What about utf8-encoding?
Is it possible to the attacker to pass a dot traversal string as utf8-encoded string, which would bypass
my current sanitizing function?


Let's find out!


From the previous test at the very top of this document (in the so-called teaser part), I know that
my file encoding is UTF8-C form (because "\xC3\xA1" === "á"). Your mileage may vary, but here
I'll be focusing on that specific form of utf8.


According to this utf8 page: https://en.wikipedia.org/wiki/UTF-8,
the dot traversal chars in utf8 would be:
 
- 002e (the dot)
- 002e
- 002f (the slash)

In php, we can use utf8 code as string literals as of php7.0, by using the \u{XXX} notation.
Source: https://www.php.net/manual/en/language.types.string.php

So what if we pass it via the url?

```txt
http://myapp?t=\u{002e}\u{002e}\u{002f}test
```

We obtain this $_GET array:

```html
array(1) {
  ["t"] => string(28) "\u{002e}\u{002e}\u{002f}test"
}

```


So the dot traversal is passed via the uri in this form, but is it harmful?

Let's try to include it directly:

```php
$dotTraversal = $_GET['t'];
$file = $this->confDir . "/" . $dotTraversal . ".byml";
a(file_exists($file)); // false
```

The file doesn't exist, and in fact, the utf8 chars are not interpreted as utf8.

I suppose that it's because the result of urldecode is seen as a single quoted string, not a double quoted one,
but I'm not 100% sure.

And therefore, since there is another safer technique, let's use the safer technique and avoid the doubt.

However I posted a question about that:
https://stackoverflow.com/questions/58395670/php-get-is-it-interpreted-as-single-quoted-or-double-quoted-string

and people tends to think that the variables from a $_GET array are just non-interpreted strings (in other words
they won't be interpreted as double quoted strings).

But anyway there is a safer method, so let's do the safest method.





The safest technique
-------------


Source: https://stackoverflow.com/questions/4205141/preventing-directory-traversal-in-php-but-allowing-paths
```php

$basepath = '/foo/bar/baz/';
$realBase = realpath($basepath);

$userpath = $basepath . $_GET['path'];
$realUserPath = realpath($userpath);

if ($realUserPath === false || strpos($realUserPath, $realBase) !== 0) {
    //Directory Traversal!
} else {
    //Good path!
}
```



 

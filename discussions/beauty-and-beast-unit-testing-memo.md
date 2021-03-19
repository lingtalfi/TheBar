The beauty and the beast unit testing memo
===============
2021-03-05


Back in 2015 I wrote a unit testing system called "The beauty and the beast".

In this discussion, I write a little practical memo for myself (and other users) to get
started quickly with those tools.



First, we need to install the two components:


```bash
lt install Ling.Beauty
lt install Ling.PhpBeast
```




Then we need to create a directory where to put our tests.

I choose:

- /universe/Ling/TokenFun/bnb   (since I'm working in the TokenFun planet)



Now let's create some test files:


- /universe/Ling/TokenFun/bnb/use-statements-parser-parseTokens.bnb.php


I added the **bnb.php** extension, so that the beauty component can collect all those extensions and show them to me.


Then I put my tests in that file.


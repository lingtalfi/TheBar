Handy exception
========
2021-04-06





A "handy exception" is a php exception.

If its code is (int) 2, it means it's an input error, aka client error, aka gui error, aka user error.

In other words, the message of a **handy exception** with code 2 should be delivered to the user that provoked
the error.



Otherwise, it's a regular app exception.
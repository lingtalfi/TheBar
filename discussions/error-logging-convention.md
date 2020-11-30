The error logging convention
================
2020-11-30





This is a convention that you can use if you want.
It's mostly useful for both application maintainers and third party plugin authors.


The convention is the following:

- a log message sent to the **error** channel (aka type) indicates an error that should be fixed by the application maintainer


The main idea is to create a (todo)list of error messages that the **application maintainer** can then fix.

Often, in addition to writing the **error** log message to a file, the application maintainer will also send him/herself an email, so that
the fix can be applied at the earliest time. 







The permissive dev environment
===========
2020-06-30



The **permissive dev environment** is just a label for the php webdev development environment described below.



In the **permissive dev environment**, the web server process is running as the user owning all the files, so that
creating deleting files always work (i.e. no permission problems).


In apache for instance, it's easily done by changing the "User" and "Group" directives to your regular username and group:

```apacheconfig
#
# If you wish httpd to run as a different user or group, you must run
# httpd as root initially and it will switch.  
#
# User/Group: The name (or #number) of the user/group to run httpd as.
# It is usually good practice to create a dedicated user and group for
# running httpd, as with most system services.
#
User ling
Group staff
```



I always develop in a permissive environment, because then I don't have to think whether the file related functions will work or not.
Then, when you go to production, you have to set your folder permissions correctly. 




Basic security nugget
====================
2020-09-11





The **basic security nugget** is an array which defines which user is granted an action we're trying to execute.

It has two sections: **any** and **all**, and looks like this: 





````yaml
security:
    any:
        permission: Ling.Light_Kit_Admin.admin
        micro_permission: store.lud_user.read
    all: []          
````



In the above example, I didn't use the **all** section. In fact, I could have dropped it entirely, 
but I wanted to show you that it was there just in case.


Each section is composed of directives.

For the **any** section: if any directive passes, then the action is granted, otherwise it's denied.
For the **all** section: all directive must pass in order for the action to be granted.


As for now, the behaviour is not defined if you defined both sections at the same time, as this is a theoretical
case, not backed up by any concrete needs so far.



The available directives are:

- permission: string, the [permission](https://github.com/lingtalfi/Light_User/blob/master/doc/pages/permission-conception-notes.md) that the user must have 
- micro_permission: string, the [micro-permission](https://github.com/lingtalfi/Light_MicroPermission/) that the user must have




 


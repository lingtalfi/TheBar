Micro-permissions based system
===========
2021-06-16


Prerequisites:

- [micro-permissions](https://github.com/lingtalfi/Light_MicroPermission/blob/master/doc/pages/conception-notes.md)
- [the basic permission system](https://github.com/lingtalfi/Light_User/blob/master/doc/pages/permission-conception-notes.md)





In the [Light](https://github.com/lingtalfi/Light) framework, we have a standard permission system, with two main actors:

- **permission groups**
- **permissions**


**Permission groups** are just container for **permissions**, and the user privileges are tested with the [hasRight method of the Light_User](https://github.com/lingtalfi/Light_User/blob/master/LightUserInterface.php).



When the **micro-permissions** came out, the possibility of packing multiple **micro-permissions** into one **permission** was born (although that wasn't the original intent).


Micro-permissions are tested using the [hasMicroPermission method of the Light_MicroPermission service](https://github.com/lingtalfi/Light_MicroPermission/blob/master/doc/api/Ling/Light_MicroPermission/Service/LightMicroPermissionService/hasMicroPermission.md).



The main idea of a **micro-permissions based system** is to use exclusively **micro-permissions** to test the user privileges.

This means that we don't use the **Light_User->hasRight** method directly, but rather use only the **LightMicroPermissionService->hasMicroPermission** method.


By doing so, we place the **micro-permissions** as first class citizen, and the (regular) **permissions** become so-called **micro-permissions profiles** (aka **profile**), which are just containers for **micro-permissions**.


Plugin authors can create their own **micro-permissions profile** and be very specific about what the **profile** (i.e. permission) contains.

For instance, a blog moderator could update a post, but not create new ones, which would translate in a profile like this one:

- BlogModeratorPermission:
    - store.blog_posts.update
    - store.blog_posts.read
    

While an admin would have the whole crud rights (for instance), something like this:

- BlogAdminPermission:
    - store.blog_posts.create
    - store.blog_posts.read
    - store.blog_posts.update
    - store.blog_posts.delete
    

Or, taking advantage of the **micro-permissions** namespace notation, this is usually equivalent to this shorter syntax:

- BlogAdminPermission:
    - store.blog_posts
    


**Micro-permissions** were created originally to handle database based permissions, such as the crud example above.

Going one step further, the **micro-permissions based system** expands the scope of the **micro-permissions** to any domain, not only the database.

As we've said earlier, in this context **micro-permissions** are first class citizens.





To end this discussion, the **micro-permissions based system** is an interesting alternative to the regular permission system (in light).
It's more precise, and gives plugin authors total flexibility when creating their own profiles.


To end with a practical note: as the time of writing though, micro-permissions profiles are hard-coded by the plugin authors.

Which means the admin user cannot change them yet via a gui.

But who knows, maybe this will change someday, when the need for it will be required.







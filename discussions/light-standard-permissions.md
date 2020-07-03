Light standard permissions
===========
2020-07-03




Here we define a "standard" permission convention, which [Light](https://github.com/lingtalfi/Light) plugin authors can rely upon.

 

The standard permissions are the following:


- $pluginName.admin: 
    - can administrate the plugin (i.e. has access to the tables to configure the plugin)
- $pluginName.user: 
    - can use the plugin (i.e. has access to the service provided by the plugin, but cannot administrate it)

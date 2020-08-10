Generated custom config pattern
=============
2020-08-10



I noticed that some of my tools generate config files.
When they do so, they always add the word "generated" somewhere in the file path.

For instance, the [Light_RealGenerator plugin](https://github.com/lingtalfi/Light_RealGenerator) will generate a path like this for the realform config:

- /myapp/config/data/Light_Kit_Admin_UserPreferences/Light_Realform/generated/lup_user_preference.byml



That's fine, but now what if I need to tweak the parameters in that file?

It's not safe to manually update the generated file, since it might be overwritten again by the generator.


To be compliant with this pattern, a plugin responsible for reading the config file must use the following algorithm instead:
 
- if the path to the config file contains the expression "generated", then replace it with the expression "custom" and see if a config file exists there
- if not, use the original config file path
 
 
 
 

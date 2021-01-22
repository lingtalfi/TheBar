Import install 
================
2021-01-22



The concept of **import** and **install** has gained a lot of hype lately in the universe.

In this document, I'll explain the differences between those.



Summary
--------
2021-01-22


- **import**: imports a planet along with its dependencies, recursively. Optionally, can install the [assets/map](https://github.com/lingtalfi/UniverseTools/blob/master/doc/pages/conception-notes.md#the-planets-and-assetsmap).
- **install**: this is a two steps process:
    - **import** the planet with **assets/map**
    - if it's a light plugin, do a **logic install** (if the plugin is installable)
- **logic install**: this is a term created by the [Light_PluginInstaller](https://github.com/lingtalfi/Light_PluginInstaller) to characterize its own install operation (which allows plugins to install/update tables in a database, amongst other things)
    
        



Chronological explanation
--------
2021-01-22



Soon after the universe was first created came the [uni tool](https://github.com/lingtalfi/universe-naive-importer), 
which allowed us to **import** planets.


The novelty with the **import** concept was that it could import planet dependencies recursively, thus saving a lot of time
compared to the manual method of copy/pasting the planet dirs manually.


In addition to that, **uni tool** also provided a mechanism called [assets/map](https://github.com/lingtalfi/UniverseTools/blob/master/doc/pages/conception-notes.md#the-planets-and-assetsmap),
which allows a planet to copy files to the target application just after it's imported.

So at this point in time we have this concept of **importing** planets, with the option of copying the **assets/map** to the application.


Later, with the advent of the [light](https://github.com/lingtalfi/Light) planet, some new concepts were needed.

Light being a web oriented framework, some of its plugins required to install tables in a database, and so light created the concept of **installing**
planets, rather than just importing them.


The **install** operation starts with a regular **import** operation, but goes one step further: it also calls the light plugin's own installation method (if it has one), which is called the **logical install**.

In other words: 

- install = import + logical install 


Note: not all light plugins have a **logical install**, that's because they don't need it (i.e. they don't use a database table)



So to recap, the **uni tool** brought the concept of **import**, and light brought the concepts of **install** and **logical install**.







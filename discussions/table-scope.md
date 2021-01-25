Table scope
===========
2021-01-25


The **table scope** represents the ensemble of the tables which a plugin can create/delete, when installing itself in a database.

It serves as a safety net to avoid accidentally deleting tables from other plugins. 


In practise, it can be one of the following:

- null: this is the default. The scope should be guessed from the [create file](https://github.com/lingtalfi/TheBar/blob/master/discussions/create-file.md) of the plugin,
    which is generally the ensemble of tables which start with the first [table prefix](#table-prefix) found in the **create file**.
    
- an array: the explicit array of tables representing the **table scope** for this plugin.


The **table scope** is a concept first defined by the [Light_DbSynchronizer](https://github.com/lingtalfi/Light_DbSynchronizer/) plugin.
We make it an "official" definition, to help plugin authors.






Table prefix
--------
2021-01-25


The **table prefix** is the first underscore separated component found in the table name.

So for instance if the table name is: lud_user.

Then the **table prefix** is **lud**.


The **table prefix** acts as a namespace for plugins in a database environment.





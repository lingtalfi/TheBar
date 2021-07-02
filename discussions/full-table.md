Full table
============
2021-07-02


A **full table** is a string representing a database table with proper escaping.


In mysql, all the followings are potential full table names:


- just_a_table 
- the_db.just_a_table 
- the_db.`just_a_table` 
- `the_db`.`just_a_table` 
- `the_db`.just_a_table 


The developer providing a **full table** name to a function/method will know if/when to escape the database/table identifiers.


For instance, if the database name is "a", and the table name is "a.cor", then the developer will provide the following **full table**:

- `a`.`a.cor` 


In general, if you don't want to overthink it, just escape both the database identifier (if you have one) and the table name with backticks.
This will work all the time.







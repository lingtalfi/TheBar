Form tricks: the multiplier 
=============
2019-12-03



The multiplier is a technique we can use to insert/update multiple rows in a database at once.

It was first created to make more efficient editing for **has** tables (aka many to many table).

So for instance imagine we have the three tables: **user**, **permission_group**, and **user_has_permission_group**.

And now we want to edit the **has** table, which in this case is the **user_has_permission_group** table.

With the multiplier, we can insert and/or edit multiple rows at once.

 

How does it work?

Consider a row that you want to insert/update, like this one:

```text
- user_id: 5
- group_id: 
    - 5
    - 9
    - 15
- status: ok

```

One entry of that row is defined as the multiplier column, and it must be an array.

In our example, this would be the **group_id** column.

Then we loop through the items of the multiplier column, and basically all other entries (of the row) remain the same.

So with our example row, if we were to insert it via a multiplier compliant medium, we would end up with the following records in the database:

-
    - user_id: 5
    - group_id: 5
    - status: ok
-
    - user_id: 5
    - group_id: 9
    - status: ok
-
    - user_id: 5
    - group_id: 15
    - status: ok
 


That's for inserting rows. 

The multiplier trick works only for forms in insert mode, it wouldn't work well with forms in update mode, because it would 
be confusing as far as which value for the potential extra column (the "status" column in our example). 




The behaviour of the multiplier technique is defined by the following array:

- multiplier:
    - column: string, the name of the multiplier column
    
   
    
    
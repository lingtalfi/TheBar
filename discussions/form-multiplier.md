Form tricks: the multiplier 
=============
2019-12-03



The multiplier is a technique we can use to insert multiple rows in a database at once.

It was first created to make more efficient editing for **has** tables (aka many to many table).

So for instance imagine we have the three tables: **user**, **permission_group**, and **user_has_permission_group**.

And now we want to edit the **has** table, which in this case is the **user_has_permission_group** table.




How does it work?

Consider the following row:

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
 


That's it.


Note: at first I wanted to implement the update version of the multiplier, but I found out that it confusing for the user, so I dropped it.
However, this can be done in a not confusing way using a multiple edit module, like the one implemented in phpmyadmin for instance.
 




The behaviour of the multiplier technique is defined by the following array:

- multiplier:
    - multiplier_column: the name of the multiplier column (in our example: group_id).
    - ?insert_mode: string(insert|replace) = insert. 
            The sql keyword used for inserting queries. This matters only in case of duplicate rows.
            If you try to insert a row that already exists, what do you want to do:
            
            - reject the request (insert_mode=insert)
            - replace the old record in the database with the new one (insert_mode=replace)
   
    
    
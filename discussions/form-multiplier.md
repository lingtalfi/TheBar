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
 


That's for inserting rows. But what about updating rows?
 
Consider that now we want to update the database using the following row:

```text
- user_id: 5
- group_id: 
    - 6
    - 10
    - 12
- status: ok

```

But remember that we've already created 3 rows (with group_id=5, group_id=9 and group_id=15).
So what do we do with those old rows? Shall we remove them first before creating the new ones? or shall we just ignore them?
 
In the case of a **has** table, we usually want to remove the old rows before committing the new ones.

That sounds weird, but it appears that the update operation of the multiplier is actually not well represented by 
a traditional sql update query; rather it's a two-steps process:

- delete the old rows where user_id=5
- insert the new rows


So in a nutshell that's the update with the multiplier for a **has** table.


 


The behaviour of the multiplier technique is defined by the following array:


- multiplier: string, the name of the multiplier column
- update_cleaner: array:
    - table: mixed, the name of the table used in the **deleteQuery** (see the notes below).
    - column: string, the name of the column used in the **deleteQuery** (see notes below)
    - value: mixed, the value of the column to use in the **deleteQuery** (see the notes below)
        
    
The **deleteQuery** is used to clean the records before new ones can be inserted (in an update operation, as we've seen before).
It's a sql query that looks like this:
 
- delete * from $table where $column=$value
 
Note: actually we use pdo markers to avoid sql injection ($column=:value), but the idea remains the same.    
    
    
    
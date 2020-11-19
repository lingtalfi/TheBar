Form tricks: the multiplier 
=============
2019-12-03 -> 2020-11-19



The multiplier is a technique which helps the user inserting multiple rows in a table at once, via a form.



The basic idea
----------------
2020-11-19

The multiplier technique transforms one of the control of the form into a multiple selection control, so that the user can select
multiple values at once.

When the form is posted, an entry is inserted for each value.





How does it work?
------------
2020-11-19



First, we assume that a form is always in one of two modes:

- insert
- update, where the controls are pre-filled with values


Then, we distinguish the following **modes**:

- OWN: designed for inserting only 
- HAS: designed for insert and update
- ...more cases might be added in the future, maybe


Which **mode** to use depends on the table structure.
More on that later.


We also introduce two terms:

- owner 
- multiplied 


The **owner** term will be explained in more details in the **HAS** section.
The **multiplied** term represents the column which is multiplied during the insert/update operation.




The OWN mode
------------
2020-11-19


This case applies for tables that have a foreign key, but no **owner** (see the **HAS** mode for more details).

Typically, we have this table:

- item
    - id: ai
    - user_id: fk
    - name: varchar
    
    
    

In the **item** table above, the **user_id** column is a foreign key to the **user.id** column (assuming there is a user table with an id column).
The **user_id** column is called the **multiplied** column.


In terms of form, the form looks like this:

- INSERT NEW USER FORM
    - user_id: multiple select (array: 1, 4, 6 for instance)
    - name: john
    
    
Allowing the user to insert multiple rows at once, each row has the name equal to "john" in this case.

In order to populate the **user_id** control with all the possible values of user_id, we will use the following query:

- select id from user 


This convenience makes only sense in the insert mode of the form, since the table has no **owner**.

When the form is submitted successfully, the insert queries would look something like this:

- insert into item (id, user_id, name) values ( null, 1, john ) 
- insert into item (id, user_id, name) values ( null, 4, john ) 
- insert into item (id, user_id, name) values ( null, 6, john ) 




The HAS mode
--------------
2020-11-19


The **HAS** mode applies for "has" tables.
A "has" table is a table which primary key contains only foreign keys.

A "has" table has an owner and an owned thing, which is defined by the semantic behind the table name.
For instance, if the table name is **user_has_item**, then the owner thing is the user, and the owned thing is the item.


So typically we have these tables:


- item
    - id: ai

- user
    - id: ai

- user_has_item
    - user_id: fk
    - item_id: fk
    
    
With the **HAS** mode, we define the **owner** as the column that represents the owner thing (the **user_id** column in this example),
and the **multiplied** column as the column representing the owned thing (the **item_id** column in this example).


With the **HAS** mode, we can handle both the insert and update modes of the form.

In both cases, the form would look like this:
    
- USER_HAS_ITEM FORM
    - user_id: 5
    - item_id: multiple select (array: 201, 204, 206 for instance)
    
In both cases, we need to fetch all the possible values for the **user_id** control, using the following query:

- select id from user
  
The only difference is that in update mode, we also need to fetch the values of the items already owned by the user which id is given.
So for instance if the given **user_id** is 5, then we need to perform this query:

- select item_id from user_has_item where user_id=5


In insert mode, when the form is submitted successfully, the insert queries would look something like this:

- insert into user_has_item (user_id, item_id) values ( 5, 201 ) 
- insert into user_has_item (user_id, item_id) values ( 5, 204 ) 
- insert into user_has_item (user_id, item_id) values ( 5, 206 )


In update mode, we assume that the user wants to redefine the relationships between the user thing and the item thing, and
so we first delete all the relationships, then recreate them.
In terms of sql it looks like this:

- delete * from user_has_item where user_id=5 
- insert into user_has_item (user_id, item_id) values ( 5, 201 ) 
- insert into user_has_item (user_id, item_id) values ( 5, 204 ) 
- insert into user_has_item (user_id, item_id) values ( 5, 206 )
    










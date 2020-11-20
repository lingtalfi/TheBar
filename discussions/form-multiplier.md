Form tricks: the multiplier 
=============
2019-12-03 -> 2020-11-20



The multiplier is a technique which helps the user inserting multiple rows in a table at once, via a form.



The basic idea
----------------
2020-11-19

The multiplier technique transforms one of the control of the form into a multiple selection control, so that the user can select
multiple values at once.

When the form is posted, an entry is inserted for each value.





How does it work?
------------
2020-11-19 -> 2020-11-20



First, we assume that a form is always in one of two modes:

- insert
- update, where the controls are pre-filled with values


The multiplier trick works only in insert mode.



To update multiple rows at once, the best solution is probably to use a multiple row editor.
See the [Why no update mode](#why-no-update-mode) section for more details.




Typically, we have a table like this:

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



That's it.

    


The multiplier array
----------
2020-11-19 -> 2020-11-20

As a convenience for developers, we provide a basic array representing the multiplier's intent.
Developers can refer to this array if they want to.

- multiplier: string, the name of the **multiplied** column.







Why no update mode
----------
2020-11-20

Today I just realized that the update mode I've implemented is too risky.
The update mode basically does a "delete *", followed by an "insert *".

While it can arguably be useful with "has" tables which only have the primary key and no other columns.

This is a bad idea to use this technique on a "has" table which contains other columns, let me explain why.

For instance, consider this table:

- user_has_item
    - user_id: fk
    - item_id: fk
    - position: int
    
    
Because of the position column, it's not recommended using the multiplier form trick.

If we did, all the affected records would have the same position, because that's what the multiplier trick does, it creates identical records, except for the **multiplied** column.
So, that's not what we want.

Instead, I believe one has to realize that a multiple row editor is the only correct way (afaik) to edit multiple rows at once.    



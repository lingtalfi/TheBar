Form tricks: the multiplier 
=============
2019-12-03 -> 2020-10-01



The multiplier is a technique where we use a form to insert multiple rows in a table at once.




There are some constraints:

- the table must have a [ric](https://github.com/lingtalfi/NotationFan/blob/master/ric.md) composed of at least two columns
- there can only be one multiplier per table, see the [why only one mulitplier per table](#why-only-one-multiplier-per-table) section for more details





How does it work?

Consider the **user_has_item** table:

```text
- user_id: fk
- item_id: fk
- status: tinyint

```


We need to define which column is **multiplied**, and which is the **pivot** (i.e. by what it's multiplied by).


So the multiplied column is the **item_id** (i.e. the owned thing in the has relationship), and 
the pivot is the column **user_id** (i.e the owner in the has relationship).


Now that we know that, we need to tell the form that the multiplied control, **item_id** should be of type select multiple, or an equivalent.


There are two form modes, **insert mode** and **update mode**, and the handling of the multiplier trick is different depending on that mode.


In a nutshell, in insert mode we execute an **insert ignore** statement, while in update mode we execute a **delete** statement followed by an **insert ignore** statement.


Insert mode
-----------
2020-09-17

The user fills the form, and posts it, and we receive values like this:


- user_id: 2
- item_id: 1,3,5
- status: 0


We insert the data in the table, with the **ignore** keyword, something like this:

- insert ignore into user_has_item (user_id, item_id, status) values ( 2, 1, 0 )
- insert ignore into user_has_item (user_id, item_id, status) values ( 2, 3, 0 )
- insert ignore into user_has_item (user_id, item_id, status) values ( 2, 5, 0 )


We use the **ignore** keyword because we don't know if the user already owns the item, and we don't want to spend
an extra request to check that. With the **ignore** keyword, if the user already has the item, it will just skip the request.


That's all for the insert mode.




Update mode
------------
2020-09-17


In update mode, we receive the [ric](https://github.com/lingtalfi/NotationFan/blob/master/ric.md), usually from $_GET.

So for instance we have this ric:

- user_id: 2
- item_id: 3


If we continue the above example (i.e. the one we started in the **insert mode** section above), then there are already
3 records for user_id=2 in the **user_has_item** table (item_id=1, item_id=3 and item_id=5).


The idea with the **update mode** is to pre-fill the form with the relevant values.
In the case of the multiplied column (i.e. item_id), it means we need to get the array of all items owned by the user.
We can get it using a fill query like this:

- select item_id from user_has_table where user_id=2

Notice that **user_id** is our pivot column (that's why we need a pivot column for this technique), 
and that the value 2 is passed in the ric.


Once we pass this information to the form, the user will see the items he owns (assuming he is user_id=2).


So now the user makes his update and posts the form, let's say he gives us the following data:



- user_id: 2
- item_id: 1,4,5
- status: 0
 
 
 
Because of the rules we defined at the beginning of this document, we can safely apply the following strategy:
 

- first delete all the rows owned by the user_id=2
- then insert ignore them, just like we did in **insert mode**


The **delete** query should look something like this:

- delete from user_has_table where user_id=2


Notice that again the **pivot** column and the **ric** are required here.
 
 
 






Why only one multiplier per table
----------
2020-09-15 -> 2020-10-01




The multiplier concept brings complexity to an app that implements it.

To avoid too much complexity, we keep the multiplier concept very simple for now.

This might change if the need for more than one multiplier per table is proven useful, which it hasn't yet.





    
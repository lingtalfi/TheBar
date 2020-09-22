Clever form initiative
=============
2020-09-22



Hi, in this document we establish some conventions for form rendering.

People can follow these conventions to make forms easier to work with, or not follow them and use their own systems.



The main goals of these conventions are:


- to ease hiding specific form controls programmatically








Conventions
-------
2020-09-22


- each control is either wrapped into a html element (usually div) having the following html attributes,
    or, if it's a standalone element which cannot wrap other elements (for instance an input type=hidden element), then it must have itself the following html attributes:
    - class: cfi-control 
    - data-cfi-id: an identifier representing this control uniquely in the context of this form.
        We recommend that the identifier is in lowercase, due to the fact that some browsers have a special treatment
        for uppercase letters in html attributes.
        
            
    
    
    


     
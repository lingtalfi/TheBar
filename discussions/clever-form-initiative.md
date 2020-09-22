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


- each control is wrapped into a html element (usually div) containing:
    - class: cfi-control 
    - data-cfi-id: an identifier representing this control uniquely in the context of this form.
        The identifier must be in lowercase, due to the fact that some browsers have a special treatment
        for uppercase letters in html attributes.
    
    
    


     
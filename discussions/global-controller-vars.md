Global controller vars
==========
2021-06-25



The **global controller vars** convention describes a way for a controller to share variables with its environment.




This is done by using the [Ling.Light_Vars](https://github.com/lingtalfi/Light_Vars) service, with a namespace of "controller".


Note that this technique assumes that only one controller is rendering the page (or at least that there is one main controller rendering the page).


If your system allows for nested controllers, then you have to be aware that they potentially could both declare variables using the same "controller" namespace.







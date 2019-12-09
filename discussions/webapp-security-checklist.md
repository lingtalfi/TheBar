Web application security checklist
==============
2019-12-09



This document provides a basic security check list for web apps.

Plugins/tools can refer to this document and claim that they implemented all the items of this list,
which we believe ensure a certain level of security which we like to refer to as the "minimum security" level.


Our checklist is composed of the following documents, which one must implement fully in order to be compliant
with our "minimum security" level:

- [php session security](https://github.com/lingtalfi/TheBar/blob/master/discussions/php-session-security.md)
- implement csrf protection for all forms and ajax actions. If you're using the Light framework, we recommend one of this plugins:
    - [Light_CsrfSession plugin](https://github.com/lingtalfi/Light_CsrfSession) (preferred because simpler to develop with)
    - [Light_CsrfSimple plugin](https://github.com/lingtalfi/Light_CsrfSimple) 
    - [Light_Csrf plugin](https://github.com/lingtalfi/Light_Csrf) 
    Or otherwise, some other tools might help you:
    - [CSRFTools](https://github.com/lingtalfi/CSRFTools)





 
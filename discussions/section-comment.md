Section comment
============
2020-07-21 -> 2021-03-19


A **section comment** (aka **section header comment**) is a comment that looks like this in a php file:


```php

<?php

//--------------------------------------------
// MY SECTION
//--------------------------------------------

```


The **section header comment** looks like this in a [babyYaml](https://github.com/lingtalfi/BabyYaml) file:


```yaml
# --------------------------------------
# MY SECTION
# --------------------------------------

```


A **section header comment** is basically composed of the following sequence of lines:

- a dash line
- a title line
- a dash line



A **section** is composed of the following elements:

- a **section header comment**
- the section content


**Sections** end when either of those is true:

- the file ends
- the **section header comment** of the next section starts
- a **dash line** is found



Here is an example of different sections in a babyYaml file.


```txt

# --------------------------------------
# Ling.Light_Kit_Admin
# --------------------------------------
pou
pou
pou


# --------------------------------------
# Ling.XX
# --------------------------------------
pou
dji
pou

# --------------------------------------

```



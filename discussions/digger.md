Digger convention
=========
2021-06-29

The **digger convention** is a convention used by planets to share some information with each other.

This information is generally used by automation tools
(such as [Ling.Light_DeveloperWizard](https://github.com/lingtalfi/Light_DeveloperWizard)) that needs to know specific
things about planets which are hard to guess otherwise.

The **digger convention** is simple: just create a **digger.byml** file at the root of the planet.

This **digger.byml** file is a [babyYaml](https://github.com/lingtalfi/BabyYaml) file containing the information you
share.

As an example, here is what the [Ling.Light_Kit_Store](https://github.com/lingtalfi/Light_Kit_Store) planet shared as of 2021-06-29 in its **digger.byml** file:


```yaml
route_prefix: lks_route-
```








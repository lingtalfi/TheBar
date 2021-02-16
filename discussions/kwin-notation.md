Kwin notation
=============
2021-02-16

Kwin notation is a way of writing documentation for cli commands.

I use this notation because then, using third party tools, I can generate php methods (for
the [LightCliCommandInterface](https://github.com/lingtalfi/Light_Cli/blob/master/doc/api/Ling/Light_Cli/CliTools/Program/LightCliCommandInterface.md))
from my markdown file directly, thus saving me some time.

Kwin notation describes commands.

Each command's notation should be as follows:

```txt
- **$commandName**: $description
$argumentDescription?
```

With:

- **$commandName**: the name of the command (only alpha-numeric chars)
- **$description**: the description of the command, using [mini markdown notation](#mini-markdown).
- **$argumentDescription**: the argument description, as explained below



### mini markdown
2021-02-16


The mini markdown notation only is just regular text, but the double asterisk wrapping (which makes a word bold in markdown)
is allowed.

it can span over multiple lines.



### argument description
2021-02-16


The argument description is like this (the number of indenting whitespace is not double-checked by my third party tools, 
so it's not very important):


```txt
    - Arguments:
      - parameters: 
        - $parameterName: $parameterDefinition 
        - ... 
      - options: 
        - $optionName: $optionDefinition
        - ... 
      - flags: 
        - $flagName: $flagDefinition
        - ... 
```

All definitions can use [mini markdown notation](#mini-markdown).



Example
----------
2021-02-16


In the following example, we have two commands defined (help and import):

```txt
- **help**: displays the help 
- **import**:
    With no argument, reads the [lpi.byml](#the-lpibyml-file) file and makes sure every planet defined in it is imported (along with its dependencies, recursively) in the host app.
    The [assets/map](https://github.com/lingtalfi/UniverseTools/blob/master/doc/pages/conception-notes.md#the-planets-and-assetsmap) are not copied.
      
    - Arguments:
      - parameters: 
        - planetDefinition: if the **planetDefinition** argument is defined, it will [import](https://github.com/lingtalfi/TheBar/blob/master/discussions/import-install.md#summary) 
          the given planet (and its dependencies recursively), without the **assets/map**, and update the **lpi.byml** file accordingly, using a plus symbol at the end of every newly imported planet's version number.
          
            The **$planetDefinition** stands for: $planetDotName(:$versionExpression)?
          
            With:
            - planetDotName: the [planetDotName](https://github.com/karayabin/universe-snapshot#the-planet-https://github.com/lingtalfi/TheBar/blob/master/discussions/import-install.md#summarydot-name)
            - versionExpression: the [versionExpression](#version-expression), defaults to last if not defined
      - options: 
        - bernoni: string (auto|manual) = auto. The mode to use when a [bernoni conflict](#the-bernoni-problem-what-happens-in-case-of-conflict) occurs.
      - flags: 
        - keep-build: if set, the [build dir](#importing-to-the-build-dir) will not be automatically removed after a successful operation.
        - d: if set, enables the debug mode, in which all log levels messages are displayed
        - n: if set, doesn't update the **lpi file** when the **planetDefinition** parameter is defined
        - f: if set, forces the reimporting of the planet, even if it's already in your app
```







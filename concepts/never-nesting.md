---
description: >-
  Never nesting is a concept to write code that's extremely helpful for
  readability
icon: nesting-dolls
---

# Never Nesting

Never Nesting is a concept to write code with high readability.

Have you ever came across a script like this?

```applescript
command /pay <player> <number>:
    trigger:
        if arg-1 is not player:
            if arg-2 > 0:
                if player's balance is greater than or equal to arg-2:
                    remove arg-2 from player's balance
                    add arg-2 to arg-1's balance
                    send "&aYou paid %arg-1% $%arg-2%."
                    send "&aYou received $%arg-2% from %player%." to arg-1
                else:
                    send "&cYou don't have enough money!"
            else:
                send "&cYou must pay a positive amount!"
        else:
            send "&cYou can't pay yourself!"
```

So many indentations! Finding ending conditions for scripts like this (especially longer ones) makes reading scripts quite difficult.

One thing we can do to avoid this kind of programming is by using Never Nesting

## What is Never Nesting?

Never Nesting is the idea of extracting and inverting conditions

Instead of checking whether a condition is true, check if the condition is false, then handle your 'failing' or what would be in your else block first.

That would take a condition that looks like this:

```applescript
if {_entity} is a parrot:
    send "You just right clicked a parrot!"
else:
    send "This is not a parrot!"
```

To work like the following:

```applescript
if {_entity} is not a parrot:    # Make sure everything needed for the rest of the  
    send "This is not a parrot!" # skript is true before continuing
    stop
send "You just right clicked a parrot"
```

While this may not look like it changes much, it's incredibly helpful when the sizes of scripts increases and gain more conditions.

The example script could begin with a section `(area of code, not Skript's section syntax)` of code with just error handling, making both the conditions necessary for the script's functioning and their corresponding errors easily apparent.

```applescript
command /pay <player> <number>:
    trigger:
        # Handle any cases we don't want happening with their errors
        if arg-1 is player:
            send "&cYou can't pay yourself!"
            stop
        if arg-2 <= 0:
            send "&cYou must pay a positive amount!"
            stop
        if player's balance < arg-2:
            send "&cYou don't have enough money!"
            stop
        # Continue with our code
        remove arg-2 from player's balance
        add arg-2 to arg-1's balance
        send "&aYou paid %arg-1% $%arg-2%."
        send "&aYou received $%arg-2% from %player%." to arg-1
            
```

When looking at the first iteration of the script, all of the errors are all the way on the other side of the script to their conditions. This makes it more difficult to identify where exactly some conditions point to.

## What if I need to keep nesting?

Nesting is inevitable, but a way we can prevent having a single script with lots of nesting is by using functions. When indentations get too much, extract the parts of the script that are too heavily nested and put it into it's own function. This may also allow for other areas of your server to use this function `(more on that in the abstraction section)`&#x20;

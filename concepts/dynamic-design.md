---
description: >-
  This page shows how to use dynamic functions, the new experiment to make
  highly scalable systems
icon: ruler-triangle
---

# Dynamic Design

### What is this?

**Dynamic design** is a method in which you can create highly reusable systems. This comes in very useful for things like magic, fighting games, enemy types as those often require "templates", such as describing what a base enemy can have or do. With dynamic design, we can pull away these elements and "soft code"

### Lets make a spell.

Say we have 3 spells:

* A poison spell  (poison enemies)
* A fire spell, (lights enemies on fire)
* A ice spell,  (freezes enemies)

Here is the code, that most people might write to achieve this task.

```applescript
on right click with a blaze rod:
    cast_spell(player)

function cast_spell(p: player):
    set {_velocity} to vector in direction of {_p}

    # Choose what spell to cast
    if metadata "poison" of {_p} = true:

        shoot arrow from {_p}:
            set velocity of projectile to {_velocity}
            set metadata "posion" of projectile to true
        broadcast "Spell of POSION cast!"

    else if metadata "fire" of {_p} = true:

        shoot arrow from {_p}:
            set velocity of projectile to {_velocity}
            set metadata "fire" of projectile to true 
        broadcast "FLAMES! BURST!"

    else if metadata "ice" of {_p} = true:

        shoot arrow from {_p}:
            set velocity of projectile to {_velocity}
            set metadata "ice" of projectile to true
        broadcast "Icy..."

on projectile hit:
    # Choose which effects to apply on hit
    if metadata "poison" of projectile = true:
        poison victim for 3 seconds

    else if metadata "fire" of projectile = true:
        set fire to victim for 3 seconds

    else if metadata "ice" of projectile = true:
        set freeze time of victim to 3 seconds
```

* It repeats the same `shoot arrow` effect for each spell type.
* Adding a new spell means copy-pasting an `else if` section in 2 different places, becoming slowly tedious over time.
* If you ever want to change how projectiles are shot (velocity, particles, effects), you have to do it in multiple places.

### Why should we switch to dynamic design?

The roots of **dynamic design** are by asking questions such as:

_“What if we wanted to add a lightning spell? What would we have to do?”_

And understanding how that is a hassle for the developer to expand their project, if we follow this project architecture. In order to create a better system for creating spells, we can **modularize** the spells.

### Modularization.

Lets take out each spell and put it into its own file, `fire.sk`, `poison.sk` and `ice.sk`, then call what we just wrote `Main.sk`.

```
scripts/
  spells/
    fire.sk
    poison.sk
    ice.sk
  Main.sk
```

How would you _efficiently_ deal with all of the logic for each spell in its own separate file, trying to touch `Main.sk` the least when creating a new spell?

{% code title="Main.sk" %}
```applescript
using script reflection

on right click with a blaze rod:
    cast_spell(player)

function cast_spell(p: player):
    set {_velocity} to vector in direction of {_p}

    # Get the spell type from a metadata tag containing a spell id, eg.
    # poison, fire, etc.
    set {_spell} to metadata "spell_type" of {_p}

    # Shoot our spell
    shoot arrow from {_p}:
        set velocity of projectile to {_velocity}
        # Set the spell type of projectile to the spell id
        set metadata "spell_type" of projectile to {_spell}
        set {_projectile} to projectile

    # Trigger our spell's spawn "event" with our projectile
    set {_function} to function "on_spawn" in (script "spells/%{_spell}%")
    run {_function} with arguments ({_projectile})

on projectile hit:
    if victim is not set:
        stop

    set {_spell} to metadata "spell_type" of projectile

    # Trigger our spell's hit "event", with our projectile and victim 
    set {_function} to function "on_hit" in (script "spells/%{_spell}%")
    run {_function} with arguments ((projectile) and (victim))
```
{% endcode %}

{% code title="spells/fire.sk" %}
```applescript
local function on_spawn(projectile: entity):
    broadcast "FLAMES! BURST!"

local function on_hit(projectile: entity, victim: entity):
    set fire to {_victim} for 3 seconds
```
{% endcode %}

{% code title="spells/poison.sk" %}
```applescript
local function on_spawn(projectile: entity):
    broadcast "Spell of POSION cast!"

local function on_hit(projectile: entity, victim: entity):
    poison {_victim} for 3 seconds
```
{% endcode %}

{% code title="spells/ice.sk" %}
```applescript
local function on_spawn(projectile: entity):
    broadcast "Icy..."

local function on_hit(projectile: entity, victim: entity):
    set freeze time of {_victim} to 3 seconds
```
{% endcode %}

* `Main.sk` doesn't need to know what spells exist.
* New spells are just new files with a `on_hit()` and `on_spawn()` function.&#x20;
* Each file handles only its own concern.
* Only one spot for casting/spell management.

If we now want to add a "ticking function" for each projectile's lifetime, we can add that super easily.\
A local function in each spell named `on_tick()`, then every tick we execute that for every projectile.

{% code title="Main.sk" %}
```applescript
using for loops
# ...

function cast_spell(p: player):
    set {_velocity} to vector in direction of {_p}
    set {_spell} to metadata "spell_type" of {_p}

    shoot arrow from {_p}:
        set velocity of projectile to {_velocity}
        set metadata "spell_type" of projectile to {_spell}
        set {-projectiles::%projectile%} to projectile
# ...

every tick:
    for {_projectile} in {-projectiles::*}:
        if {_projectile} is not ticking:
            delete {-projectiles::%{_projectile}%}
            continue

        set {_spell} to metadata "spell_type" of {_projectile}
        set {_tick_function} to function "on_tick" in (script "spells/%{_spell}%")
        if {_tick_function} is not set:
            continue

        run {_tick_function} with arguments ({_projectile})
```
{% endcode %}

{% code title="spells/fire.sk" %}
```applescript
local function on_tick(projectile: entity):
    draw flame at {_projectile}
```
{% endcode %}

Through this we are easily able to add extensible features to each projectiles lifetime, kind of like events for each spell type.\
\
This structure reduces duplication, improves organization, and makes expanding your games _so much easier_. Have a look at the docs for script reflection and experiment with other uses, like making an upgrade system, creation is the only way you will understand how to use this to your full ability.

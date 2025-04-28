---
description: >-
  A library of scripts to make your game feel more complete, and have more
  visual info.
icon: scroll
---

# Library of useful scripts

## A nice name-tag base.

Generally you would put some information about your player's data here, in this case its their name and health, but this could be used to show how far they are on a leader board, etc.

{% code title="Nametag.sk" overflow="wrap" %}
```applescript
# Dependencies: SkCheese (only used for string creation, can be replaced.)
# -- General config --

options:
    scale: vector(0.9,0.9,0.9) 
    translation: vector(0,0.2,0)  # A bit above the head
    background: rgb(20,20,30,100) # Slight blueish gray
    text_shadow: true

function get_tag_format(p: player) :: string:
    # Shows name and the health of the player
    new string joined with nl stored in {_string}:
        " %{_p}% "
        " %health of {_p}% "
    return formatted ({_string})

# -- Events --
on script load:
    refresh_tag(all players)

on join:
    refresh_tag(player)

on leave:
    delete_tag(player)

on death:
    delete_tag(victim)

on respawn:
    refresh_tag(player)

on gamemode change:
    if (gamemode of player) is spectator:
        delete_tag(player)
    else:
        refresh_tag(player)

# -- Modification functions --

# I would reccomend updating this in your secondary ticking function
# running every 1s, but it depends on the info that is being displayed
function update_tag(p: player):
    set display text of (metadata "tag" of {_p}) to get_tag_format({_p})

function create_tag(p: player):
    spawn text display at block at {_p}:
        set display text of entity to get_tag_format({_p})
        set display scale of entity to {@scale}
        set display translation of entity to {@translation}
        set display text shadowed of entity to {@text_shadow}
        set color of entity to {@background}
        set billboard of entity to vertical
        set metadata "tag" of {_p} to entity 
    make (metadata "tag" of {_p}) ride {_p}
    hide (metadata "tag" of {_p}) from {_p}

function refresh_tag(p: players):
    loop {_p::*}:
        delete_tag(loop-value)
        create_tag(loop-value)

function delete_tag(p: player):
    kill metadata "tag" of {_p}
    delete metadata "tag" of {_p}
```
{% endcode %}

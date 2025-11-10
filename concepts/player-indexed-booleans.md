---
description: Why Player Indexed booleans are bad.
icon: check
---

# Player Indexed Booleans

## A game of Tag.

Assume we have a game of tag, and we have a function to tag players like this:

{% code overflow="wrap" lineNumbers="true" fullWidth="false" %}
```applescript
function tagPlayer(player: player):
  set {taggedPlayers::%{_player}%} to true
```
{% endcode %}

The basic datastructure we have here is `{list::%player%} = boolean`, where the player turns into a uuid. This is a naive approach to a 'flag' in Skript.

This look inconspicious at first, a way to get a boolean from a player, but all values are nullable in Skript, so our datastructure looks a lot more like `{list::%player?%} = boolean?` .

The correct way to do this in Skript would be `{list::%player?%} = player?`&#x20;

{% code overflow="wrap" lineNumbers="true" %}
```applescript
function tagPlayer(player: player):
  set {taggedPlayers::%{_player}%} to {_player}
```
{% endcode %}

The problem with `boolean?` is that we can already infer a boolean from the nullability of `player?`, since all values are `object?`

This effectively makes `boolean` a TriState, false, true and none which we do not need if all players are untagged by default. This would just cause more bugs as we cannot _easily_ determine whether a player is untagged or tagged

This also lets us perform operations on the sets that would cost us otherwise

e.g

```applescript
send actionbar "You are tagged!" to {taggedPlayers::*}
```

whereas with `{list::%player%} = boolean` the smart way to send an actionbar to all tagged players would be... this:

{% code overflow="wrap" lineNumbers="true" %}
```applescript
send actionbar "You are tagged!" to ((indexes of {taggedPlayers::*}) mapped with [input parsed as player])
```
{% endcode %}

You can see the difference.

To remove a player, we can simply

{% code overflow="wrap" lineNumbers="true" %}
```applescript
function untagPlayer(player: player):
    delete {taggedPlayers::%{_player}%}
```
{% endcode %}

> This also removes the issue of having taken up variable storage for a boolean that we can infer.

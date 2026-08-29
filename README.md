# gbs-ActorByIndexPlugin

**Version 4.3.0. Requires GB Studio 4.3.0 or newer.**

Gives you a copy of every stock actor event that takes the actor as a number rather than from the
actor dropdown. The number can be typed in, held in a variable, or worked out by a calculation,
which means a script can decide which actor it is acting on while the game runs.

That solves two problems the dropdown cannot. A script attached to a button press or a collision
has no actor of its own, so the dropdown has nothing useful to offer. And a loop that should touch
every actor in the scene needs a number it can count up.

---

## Table of Contents

1. [Concepts](#concepts)
2. [Project Setup](#project-setup)
3. [Size Limits and Restrictions](#size-limits-and-restrictions)
4. [Events Reference](#events-reference)
5. [FAQ](#faq)
6. [Memory Footprint](#memory-footprint)
7. [Bank 0 (HOME) Usage](#bank-0-home-usage)
8. [Changelog](#changelog)

---

## Concepts

### Actor numbers

Actors are numbered from 0, in the order they appear in the scene. **0** is always the player.
The scene's own actors follow from 1, in the order you created them.

Every event here takes its actor as a value, so you can give it:

- a **number**, such as 0 for the player;
- a **variable**, global, local or temporary, holding a number;
- an **expression** that works one out.

### The attached script problem

Custom scripts let you write a sequence once and use it in many places. When one is attached to a
button or a collision, it runs with no actor attached, so the stock actor dropdown inside it has no
way to mean "the actor this belongs to".

Capture the number once with **Actor Get Index To Variable**, then hand that variable to any **By
Index** event and act on the same actor from anywhere.

### Choosing actors while the game runs

Because the actor is a value, these become straightforward:

- **Walk through every actor** by counting a variable up inside a loop.
- **Remember an actor** in a global variable early in a scene and use it much later from an
  unrelated script.
- **Pass an actor between custom scripts** as a plain number.

---

## Project Setup

1. Add **Actor Get Index To Variable** somewhere you do have a real actor, usually an actor's
   **On Init** or a custom script that receives the actor as an argument, and store the number.
2. From any other script, call the matching **By Index** event with that variable.

### Giving an attached script a handle on its own actor

```
On Init:
  Actor Get Index To Variable
    Actor:    $self$
    Variable: ActorIdx

  [attach this script to a button or a collision]

Attached script:
  Actor Set Position By Index
    Actor Index: ActorIdx
    X: 5, Y: 5
```

### Looping over every actor in a scene

```
Set Variable ActorIdx = 0
Repeat 4 times:
  Actor Hide By Index
    Actor Index: ActorIdx
  Math: ActorIdx += 1
```

### A shared effect actor

The [ContinuousScenePlugin example project](https://github.com/Mico27/gbs-ContinuousScenePlugin)
uses this plugin for a Pokémon-style tree-cutting effect. One **ActorEffect** actor is reused for
the animation, triggered by an **Attach Script to Input** event, which has no actor of its own.

https://github.com/user-attachments/assets/e5eac997-8537-4cd3-9546-1c5014afcae1

<img width="768" height="1842" alt="ActorByIndexScript" src="https://github.com/user-attachments/assets/9538f699-17f1-4aa2-9fd8-04da15fdbab5" />

A custom script called once when the scene loads receives `ActorEffect` as an argument and stores
its number:

```
Actor Get Index To Variable
  Actor:    ActorEffect   (custom script argument)
  Variable: ActorEffectIdx
```

Every time the player presses the action button facing a tree, the attached script reuses it:

```
1. Actor Set Position By Index
     Actor Index: ActorEffectIdx
     X: PlayerX,  Y: PlayerY      put it on the tree's tile

2. Actor Set Animation State By Index
     Actor Index: ActorEffectIdx
     State: default,  Loop: false play the cut animation once

3. Actor Activate By Index
     Actor Index: ActorEffectIdx  show it

   [animation plays]

4. Actor Deactivate By Index
     Actor Index: ActorEffectIdx  hide it again
```

The custom script argument is out of reach from the attached script. Storing the number in a global
variable gives that script a handle it can use whenever it likes.

The same pattern covers any small set of shared effect actors: explosions, sparks, damage numbers,
anything repositioned and replayed on demand.

---

## Size Limits and Restrictions

- A number that matches no actor in the current scene gives unpredictable results. Check numbers
  you calculate.
- Numbers belong to a scene. A variable holding one is meaningful only while that scene is loaded.
- **0** is the player. The scene's own actors start at 1.

---

## Events Reference

All events are in the **Actor** group. Events that read or write variables also appear under
**Variables**, and events that branch also appear under **Control Flow**.

Every event takes an **Actor Index** value in place of the stock actor dropdown, plus an **Other
Actor Index** where two actors are involved. Every other field matches the stock event it mirrors.

### Identity

| Event | Description |
|---|---|
| **Actor Get Index To Variable** | Stores an actor's number in a variable. Use it to capture a handle for later. |

### Visibility

| Event | Description |
|---|---|
| **Actor Show By Index** | Shows the actor. |
| **Actor Hide By Index** | Hides the actor. |

### Activation

| Event | Description |
|---|---|
| **Actor Activate By Index** | Starts the actor updating and drawing. |
| **Actor Deactivate By Index** | Stops it. |

### Collision

| Event | Description |
|---|---|
| **Actor Enable Collisions By Index** | Turns collision on for the actor. |
| **Actor Disable Collisions By Index** | Turns it off. |

### Movement

| Event | Description |
|---|---|
| **Actor Move To By Index** | Moves the actor to a position in tiles or pixels, with the usual collision and direction options. Waits until it arrives. |
| **Actor Move Relative By Index** | Moves the actor by an offset, with the same options. Waits until it arrives. |
| **Actor Move Cancel By Index** | Stops a move in progress. |
| **Actor Set Position By Index** | Puts the actor at a position immediately, in tiles or pixels. Does not wait. |
| **Actor Set Position Relative By Index** | Shifts the actor by an offset immediately. Does not wait. |

### Direction

| Event | Description |
|---|---|
| **Actor Set Direction By Index** | Sets which way the actor faces, from a fixed direction or a variable. |
| **Actor Get Direction By Index** | Reads the facing direction into a variable. |

### Properties

| Event | Description |
|---|---|
| **Actor Set Movement Speed By Index** | Sets how fast the actor moves. |
| **Actor Set Animation Speed By Index** | Sets how fast its animation plays. |
| **Actor Set Animation Frame By Index** | Jumps to a particular animation frame. |
| **Actor Set Animation State By Index** | Sets the animation state and whether it loops. |
| **Actor Set Sprite By Index** | Swaps the actor's sprite sheet. |
| **Actor Set Collision Box By Index** | Sets the actor's collision box: offset and size in pixels. |

### Reading position

| Event | Description |
|---|---|
| **Actor Get Position By Index** | Reads the actor's position into two variables, in tiles or pixels. |

### Scripts

| Event | Description |
|---|---|
| **Actor Start Update Script By Index** | Starts the actor's update script. |
| **Actor Stop Update Script By Index** | Stops it. |

### Effects

| Event | Description |
|---|---|
| **Actor Emote By Index** | Shows an emote above the actor. |
| **Actor Effects By Index** | Applies **Flicker** for a given time, **Split In** or **Split Out** with a distance and speed. |

### Projectiles

| Event | Description |
|---|---|
| **Launch Projectile Slot By Index** | Fires a preloaded projectile slot, 1 to 5, from the actor, with the same direction options as the stock event. |

### Branching

| Event | Description |
|---|---|
| **If Actor At Position By Index** | Branches on whether the actor is at a position, in tiles or pixels. |
| **If Actor Direction By Index** | Branches on which way the actor is facing. |
| **If Actor Distance From Actor By Index** | Branches on the distance in tiles between two actors, both given as numbers, using any comparison. |
| **If Actor Relative To Actor By Index** | Branches on whether one actor is above, below, left of or right of another. |

---

## FAQ

**My attached input script cannot pick the actor it belongs to. Is this the fix?**
Yes. Store the actor's number with **Actor Get Index To Variable** in its On Init, then use the
By Index events in the attached script.

**How do I hide every actor in a scene at once?**
Set a variable to 0, then repeat **Actor Hide By Index** with that variable, adding 1 each time,
for as many actors as the scene has.

**Which number is the player?**
0. The scene's own actors start at 1, in the order you created them.

**How do I find an actor's number?**
Use **Actor Get Index To Variable** with the actor picked normally. Doing it in that actor's On Init
with `$self$` is the usual way.

**Can I keep an actor number across a scene change?**
The variable survives, but the number means nothing in the new scene, because numbering is per
scene. Capture it again after the scene loads.

**What happens if the number is wrong?**
Nothing checks it, and the results are unpredictable. Keep calculated numbers inside the range of
actors the scene has.

**Can I use one effect actor for explosions all over the scene?**
Yes, and that is a common use. Store its number once, then reposition, set the animation and
activate it whenever you need the effect.

**Does it cost ROM?**
Only for the events you use, a few bytes each, the same as any built-in event. There is no fixed
cost.

**Does it clash with other plugins?**
No. It ships no engine code at all.

**Can I pass an actor into a custom script?**
Yes. Pass the number as a normal variable argument and use the By Index events inside.

---

## Memory Footprint

This plugin ships no engine code, so it has no fixed cost. Measured against the stock GB Studio
**4.3.0-e1** engine, report of 2026-08-13.

- **Bank 0, WRAM, SRAM:** nothing.
- **ROM:** the events compile a few bytes of script per call into your project, and only for the
  events you use.

---

<!-- BANK0:BEGIN -->
## Bank 0 (HOME) Usage

Bank 0 is the 16 KB fixed ROM bank shared by the GB Studio engine core, the
interrupt handlers and the GBDK runtime. Extra banked ROM is cheap to add,
bank 0 is not, so bank 0 is usually the first thing a project runs out of.

| | Bytes |
|---|---|
| Bank 0 used by this plugin | **0** |

**This plugin costs nothing in bank 0.** Everything it adds is compiled into a
switchable ROM bank.
<!-- BANK0:END -->

## Changelog

Grouped by the date each change was merged into the official
[gb-studio-plugins](https://github.com/gb-studio-dev/gb-studio-plugins) repository.

Only bug fixes, new features and feature changes are listed. Engine version bumps, patch
regeneration, packaging fixes and documentation edits are omitted.

### 2026-06-21

- Initial release, with 28 actor-by-index events.
- Added **Actor Get Index To Variable**, which writes straight to the variable and uses a temporary
  only for indirect references.
- Added **Launch Projectile In Slot By Index**.

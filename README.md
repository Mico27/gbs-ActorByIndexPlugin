# gbs-ActorByIndexPlugin

**Version 4.3.0 — Requires GB Studio ≥ 4.3.0**

Re-exposes every standard GB Studio actor event with an **index-based actor selector** instead of the built-in actor picker. The actor to operate on is supplied as a value field, so it can be a literal number, a variable, or any expression — resolved at runtime rather than baked in when the project is built.

---

## Table of Contents

1. [Concepts](#concepts)
2. [Project Setup](#project-setup)
3. [Size Limits and Restrictions](#size-limits-and-restrictions)
4. [Events Reference](#events-reference)
5. [Memory Footprint](#memory-footprint)

---

## Concepts

### Actor indices

Actor indices are **zero-based** and follow the order actors appear in the scene. Index **0** is the player actor at runtime; the scene's own actors follow in the order they were created.

Every event in this plugin takes its actor as a **value** field, so you can supply:

- a **literal number** — e.g. `0` for the player;
- a **variable** — any global, local or temp variable holding an index;
- an **expression** — any arithmetic or bitwise expression that evaluates to an index.

### The custom script problem

GB Studio's **custom scripts** let you define a sequence of events once and reuse it across many actors or scenes. But when a custom script is attached to an input or a collision, it runs detached from any actor context — the standard *Actor* picker inside it has no way to refer to "the actor that owns this script".

With this plugin you can capture an actor's index once with **Actor Get Index To Variable**, then pass that variable to any **By Index** event to operate on the same actor from anywhere. Without the plugin, the only alternative is writing the equivalent logic in raw GBVM assembly.

### Dynamic actor targeting

Because the actor is a value, patterns that are impossible with the standard picker become straightforward:

- **Loop over all actors** by incrementing a counter variable and calling By Index events inside the loop.
- **Store an actor reference** in a global variable early in a scene and retrieve it later from a completely unrelated script.
- **Pass actor targets across custom scripts** as numeric arguments.

---

## Project Setup

1. Add **Actor Get Index To Variable** wherever you have a real actor reference (typically an actor's *On Init*, or a custom script that receives the actor as an argument), and store the index in a variable.
2. From any other script, call the matching **By Index** event and feed it that variable.

### Example: actor self-reference in an attached script

```
On Init:
  Actor Get Index To Variable
    Actor:    $self$
    Variable: ActorIdx

  [Attach this script to a button or collision...]

Attached script body:
  Actor Set Position By Index
    Actor Index: ActorIdx   ← variable holding the index
    X: 5, Y: 5
```

### Example: loop over all scene actors

```
Set Variable ActorIdx = 0
Repeat 4 times:
  Actor Hide By Index
    Actor Index: ActorIdx
  Math: ActorIdx += 1
```

### Example: pooled effect actor

The [ContinuousScenePlugin example project](https://github.com/Mico27/gbs-ContinuousScenePlugin) uses this plugin to implement a Pokémon-style tree-cutting effect. A single dedicated **ActorEffect** actor serves as a reusable visual effect triggered by an *Attach Script to Input* event — a context that has no actor reference of its own.

https://github.com/user-attachments/assets/e5eac997-8537-4cd3-9546-1c5014afcae1

<img width="768" height="1842" alt="ActorByIndexScript" src="https://github.com/user-attachments/assets/9538f699-17f1-4aa2-9fd8-04da15fdbab5" />

A custom script called once when the scene loads receives `ActorEffect` as an actor argument and immediately captures its index:

```
Actor Get Index To Variable
  Actor:    ActorEffect   (custom script argument)
  Variable: ActorEffectIdx
```

Later, whenever the player presses the action button facing a tree, the attached input script reuses the stored index:

```
1. Actor Set Position By Index
     Actor Index: ActorEffectIdx
     X: PlayerX,  Y: PlayerY     ← teleport to the tree's tile

2. Actor Set Animation State By Index
     Actor Index: ActorEffectIdx
     State: default,  Loop: false ← play cut animation once

3. Actor Activate By Index
     Actor Index: ActorEffectIdx  ← make it visible and running

   [animation plays...]

4. Actor Deactivate By Index
     Actor Index: ActorEffectIdx  ← hide it when done
```

Even though `ActorEffect` was passed as a custom script argument at init time, that argument is not reachable from the attached input script. Capturing the index once and storing it in a global variable gives that script a handle it can use at any time.

This pattern generalises to any situation where you need a small pool of shared effect actors — explosions, sparks, text popups — that are repositioned and replayed on demand rather than placed statically in the scene.

---

## Size Limits and Restrictions

- An index that does not correspond to an actor in the current scene has undefined results — validate indices yourself if they are computed.
- Indices are **scene-relative**. A variable holding an index is only meaningful while the scene that produced it is loaded.
- Index **0** is the player; scene actors start at 1 at runtime.

---

## Events Reference

All events are found in the **Actor** event group. Events that read or write variables also appear in **Variables**; conditional events also appear in **Control Flow**.

Every event takes an **Actor Index** value field (and **Other Actor Index** where two actors are involved) in place of the stock actor picker. All other fields match the stock event they mirror.

### Identity

| Event | Description |
|---|---|
| **Actor Get Index To Variable** | Stores the scene index of a chosen actor into a variable. Use this to capture a self-reference for later. |

### Visibility

| Event | Description |
|---|---|
| **Actor Show By Index** | Shows the actor at the given index. |
| **Actor Hide By Index** | Hides the actor at the given index. |

### Activation

| Event | Description |
|---|---|
| **Actor Activate By Index** | Activates (enables update and rendering) the actor at the given index. |
| **Actor Deactivate By Index** | Deactivates the actor at the given index. |

### Collision

| Event | Description |
|---|---|
| **Actor Enable Collisions By Index** | Enables collision detection for the actor at the given index. |
| **Actor Disable Collisions By Index** | Disables collision detection for the actor at the given index. |

### Movement

| Event | Description |
|---|---|
| **Actor Move To By Index** | Moves the actor to an absolute tile or pixel position. Supports wall/actor collision and move type (horizontal, vertical, diagonal). Waits until the move completes. |
| **Actor Move Relative By Index** | Moves the actor by a relative offset, with the same collision and move-type options. Waits until the move completes. |
| **Actor Move Cancel By Index** | Cancels any in-progress movement for the actor. |
| **Actor Set Position By Index** | Instantly teleports the actor to an absolute position (tiles or pixels). Does not wait. |
| **Actor Set Position Relative By Index** | Instantly offsets the actor by a relative amount. Does not wait. |

### Direction

| Event | Description |
|---|---|
| **Actor Set Direction By Index** | Sets the facing direction. The direction can be a literal (`up`, `down`, `left`, `right`) or a variable. |
| **Actor Get Direction By Index** | Reads the facing direction into a variable. |

### Properties

| Event | Description |
|---|---|
| **Actor Set Movement Speed By Index** | Sets the actor's movement speed. |
| **Actor Set Animation Speed By Index** | Sets the actor's animation frame tick speed. |
| **Actor Set Animation Frame By Index** | Jumps the actor to a specific animation frame. |
| **Actor Set Animation State By Index** | Sets the animation state, and whether it loops. |
| **Actor Set Sprite By Index** | Swaps the actor's sprite sheet at runtime. |
| **Actor Set Collision Box By Index** | Overrides the actor's collision bounding box (offset X/Y and size W/H in pixels). |

### Position readback

| Event | Description |
|---|---|
| **Actor Get Position By Index** | Reads the actor's current position into two variables (X and Y), in tiles or pixels. |

### Scripts

| Event | Description |
|---|---|
| **Actor Start Update Script By Index** | Starts the actor's update script. |
| **Actor Stop Update Script By Index** | Stops the actor's update script. |

### Effects

| Event | Description |
|---|---|
| **Actor Emote By Index** | Plays an emote above the actor. |
| **Actor Effects By Index** | Applies a visual effect to the actor: **Flicker** (for a given duration), **Split In**, or **Split Out** (with configurable distance and speed). |

### Projectiles

| Event | Description |
|---|---|
| **Launch Projectile Slot By Index** | Launches a pre-loaded projectile slot from the actor. Uses a slot number (1–5) instead of defining the projectile inline, and supports the same direction modes as the stock event. |

### Conditionals

| Event | Description |
|---|---|
| **If Actor At Position By Index** | Branches on whether the actor is at a specific position (tiles or pixels). |
| **If Actor Direction By Index** | Branches on whether the actor is facing a specific direction. |
| **If Actor Distance From Actor By Index** | Branches on the tile distance between two actors (each given by index), using a comparison operator (`<`, `<=`, `==`, `>=`, `>`, `!=`). |
| **If Actor Relative To Actor By Index** | Branches on the spatial relationship between two actors (each given by index): is above, is below, is left of, is right of. |

---

## Memory Footprint

- **WRAM added:** 0 bytes.
- **SRAM added:** 0 bytes.
- **ROM:** small — a set of native helpers in banked ROM, plus a few bytes of GBVM script per event call in your project's script banks. Only the events you actually use are compiled in.

---

<!-- BANK0:BEGIN -->
## Bank 0 (HOME) Usage

Bank 0 is the 16 KB non-switchable ROM bank that the GB Studio engine core,
the interrupt handlers and the GBDK runtime all share. Banked ROM is cheap
(add another bank), bank 0 is not, so it is usually the first thing a project
runs out of.

| | Bytes |
|---|---|
| Bank 0 used by this plugin | **0** |
| Bank 0 free with this plugin installed | **1,451** of 16,384 (91% used) |

**This plugin costs nothing in bank 0.** All of its code lives in a switchable
ROM bank; nothing it adds is resident in bank 0.

<details><summary>How this was measured</summary>

GB Studio 4.3.2, DMG target, default engine settings. Each module's bank 0
contribution is the `A _HOME size` record that SDCC writes into its `.rel`
object, summed over the engine sources this plugin provides. Stock sizes come
from building projects whose only plugin ships no engine C, so every module in
them is the untouched engine; two such builds were compared and agreed on all
73 shared modules.

The "free" figure is a stock project with this plugin and nothing else. Your
own number will differ: other plugins, and any engine settings that change what
the core compiles, move it independently of this plugin.

</details>
<!-- BANK0:END -->

## Changelog

Grouped by the date each change was merged into the official
[gb-studio-plugins](https://github.com/gb-studio-dev/gb-studio-plugins) repository.

Only bug fixes, new features and feature changes are listed. Engine version
bumps, patch regeneration, packaging fixes and documentation edits are omitted.

### 2026-06-21

- Initial release, with 28 actor-by-index events.
- Added an "Actor Get Index To Variable" event, writing directly to the variable alias and only using a temporary for indirect references.
- Added "Launch Projectile In Slot By Index".

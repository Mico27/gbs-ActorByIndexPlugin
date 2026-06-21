# Actor By Index Plugin

**Author:** Mico27  
**Version:** 4.3.0  
**Requires:** GB Studio >= 4.3.0

Re-exposes every standard GB Studio actor event with an **index-based actor selector** instead of the built-in actor picker. The actor to operate on is supplied as a `value` field, meaning it can be a literal number, a variable, or any expression — resolved at runtime rather than baked in at compile time.

---

## Why this plugin exists

### The custom script problem

GB Studio's **custom scripts** (reusable scripts called with *Call Custom Script*) let you define a sequence of events once and reuse it across many actors or scenes. However, when a custom script is **attached to an input or collision** (via *Attach Script to Input / Actor*), it runs detached from any actor context — the standard *Actor* picker inside the script has no way to refer to "the actor that owns this script" dynamically.

With this plugin you can:

1. Use **Actor Get Index To Variable** at the start of the script to store the calling actor's scene index into a variable.
2. Pass that variable to any **By Index** event to operate on that same actor throughout the rest of the script.

Without this plugin the only alternative is writing the equivalent logic in raw GBVM assembly.

### Dynamic actor targeting

Because `actorIndex` is a `value` field, you can drive it with any variable or expression, enabling patterns that are impossible with the standard actor picker:

- **Loop over all actors** by incrementing a counter variable and calling By Index events inside the loop.
- **Store an actor reference** in a global variable early in a scene and retrieve it later from a completely unrelated script.
- **Pass actor targets across custom scripts** as numeric arguments.

---

## Example: actor self-reference in an attached script

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

---

## Example: loop over all scene actors

```
Set Variable ActorIdx = 0
Repeat 4 times:
  Actor Hide By Index
    Actor Index: ActorIdx
  Math: ActorIdx += 1
```

---

## Installation

Copy the `src/ActorByIndexPlugin` folder into your GB Studio project's `plugins/` directory:

```
your-project/
  plugins/
    ActorByIndexPlugin/
      plugin.json
      events/
        eventActorShowByIndex.js
        ...
```

An example project is included in `ActorByIndexTest/`.

---

## Events

All events are found in the **Actor** event group. Events that read or write variables also appear in **Variables**. Conditional events also appear in **Control Flow**.

### Identity

| Event | Description |
|---|---|
| **Actor Get Index To Variable** | Stores the scene index of a chosen actor into a variable. Use this at the start of an attached script to capture a self-reference. |

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
| **Actor Move To By Index** | Moves the actor at the given index to an absolute tile or pixel position. Supports wall/actor collision and move type (horizontal, vertical, diagonal). Waits until the move completes. |
| **Actor Move Relative By Index** | Moves the actor at the given index by a relative offset. Same collision and move-type options as Move To. Waits until the move completes. |
| **Actor Move Cancel By Index** | Cancels any in-progress movement for the actor at the given index. |
| **Actor Set Position By Index** | Instantly teleports the actor at the given index to an absolute position (tiles or pixels). Does not wait. |
| **Actor Set Position Relative By Index** | Instantly offsets the actor at the given index by a relative amount. Does not wait. |

### Direction

| Event | Description |
|---|---|
| **Actor Set Direction By Index** | Sets the facing direction of the actor at the given index. The direction can be a literal (`up`, `down`, `left`, `right`) or a variable. |
| **Actor Get Direction By Index** | Reads the facing direction of the actor at the given index into a variable. |

### Properties

| Event | Description |
|---|---|
| **Actor Set Movement Speed By Index** | Sets the movement speed of the actor at the given index. |
| **Actor Set Animation Speed By Index** | Sets the animation frame tick speed of the actor at the given index. |
| **Actor Set Animation Frame By Index** | Jumps the actor at the given index to a specific animation frame. |
| **Actor Set Animation State By Index** | Sets the animation state (and whether it loops) of the actor at the given index. |
| **Actor Set Sprite By Index** | Swaps the sprite sheet of the actor at the given index at runtime. |
| **Actor Set Collision Box By Index** | Overrides the collision bounding box of the actor at the given index (offset X/Y and size W/H in pixels). |

### Position readback

| Event | Description |
|---|---|
| **Actor Get Position By Index** | Reads the current position of the actor at the given index into two variables (X and Y), in tiles or pixels. |

### Scripts

| Event | Description |
|---|---|
| **Actor Start Update Script By Index** | Starts the update script of the actor at the given index. |
| **Actor Stop Update Script By Index** | Stops the update script of the actor at the given index. |

### Effects

| Event | Description |
|---|---|
| **Actor Emote By Index** | Plays an emote above the actor at the given index. |
| **Actor Effects By Index** | Applies a visual effect to the actor at the given index: **Flicker** (for a given duration), **Split In**, or **Split Out** (with configurable distance and speed). |

### Conditionals

| Event | Description |
|---|---|
| **If Actor At Position By Index** | Branches based on whether the actor at the given index is at a specific position (tiles or pixels). |
| **If Actor Direction By Index** | Branches based on whether the actor at the given index is facing a specific direction. |
| **If Actor Distance From Actor By Index** | Branches based on the tile distance between two actors (each identified by index), using a comparison operator (`<`, `<=`, `==`, `>=`, `>`, `!=`). |
| **If Actor Relative To Actor By Index** | Branches based on the spatial relationship between two actors (each identified by index): is above, is below, is left of, is right of. |

---

## The `actorIndex` value field

Every event in this plugin accepts `actorIndex` (and `otherActorIndex` where relevant) as a **value** field. This means you can supply:

- A **literal number** — e.g. `0` for the first scene actor.
- A **variable** — any global, local, or temp variable holding the index.
- An **expression** — any arithmetic or bitwise expression that evaluates to an index.

Actor indices are zero-based and correspond to the order actors appear in the scene (the `_index` field in each actor's `.gbsres` file). Index `0` is the first scene actor; the player character is not included in the scene actor list.



https://github.com/user-attachments/assets/e5eac997-8537-4cd3-9546-1c5014afcae1



<img width="768" height="1842" alt="ActorByIndexScript" src="https://github.com/user-attachments/assets/9538f699-17f1-4aa2-9fd8-04da15fdbab5" />

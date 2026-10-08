---
name: animating-frames
description: "Adds motion to the frames of a Streamloop scene through the Streamloop MCP — how layers enter and leave, how bound position and size changes tween, how changing values roll or swap (count up, flash), and how a frame transitions in. Use when someone working on a Streamloop scene says \"animate\", \"slide in\", \"fade\", \"count up\", \"stagger\", \"transition\", or wants something on their stream to move instead of cut."
---
# Animating frames

> With the Streamloop MCP: every `scene_*` tool also takes `scene`, the scene's id (`scn_…`) — a call written `scene_get('frame/intro', { as: 'image' })` is `scene_get { scene, path: "frame/intro", as: "image" }`. Changing or removing something that exists also takes its `revision` (a layer: its own or its frame's; `"*"` overwrites on purpose), or the answer is REVISION_REQUIRED. Every set changes the draft only: viewers see it after `publish_scene { scene, draftRevision }` (the `draftRevision` every answer carries; while a stream plays the scene it answers CONFIRM_REQUIRED — tell the user what changes, then `confirm: true`). Starting one from nothing: skill building-scenes.

Motion is a layer's `motion` prop plus the frame's transition. Each prop is described at scene_get('element/<Type>').

```jsx
<Stack id="strap" x={96} y={836} padding={[20, 32]} motion={{ enter: { kind: "rise", ms: 420 }, exit: { kind: "fade", ms: 200 } }} style={{ background: "#0B0F1AE6" }}>
  <Text id="strapName" value={controls.guestName} style={{ fontSize: 56, fontWeight: 800, color: "#FFFFFF" }} />
</Stack>
<Number id="homeGoals" x={96} y={120} value={state.score.home} motion={{ change: "countUp" }} style={{ fontSize: 72, fontWeight: 800, color: "#FFFFFF" }} />
<Rect id="lead" x={96} y={240} w={state.score.home * 200} h={12} motion={{ transition: { ms: 600, ease: "out" } }} style={{ background: "#E63946" }} />
```

- `enter` plays when the layer mounts: its frame goes on air, its condition turns true, its row is added. `exit` plays when it goes. Kinds: fade · rise (up from below) · drop (down from above) · slide (in from the left) · wipe · scale; `ms`, `delay`. A kind has one direction; there is no `from`.
- `motion.transition` tweens a layer's position and size changes (a bound x or w, a Stack re-flowing) instead of jumping. It is not the frame's `transition` (last point).
- `change` on a bound `value`: `swap` lifts the old text out and the new one in (480 ms), `countUp` rolls a Number to its new value (800 ms, formatted as it rolls; on a Text it does nothing — make the score a Number), `flash` pulses a highlight on change (300 ms); leave it out to cut. On air only `swap` animates; `countUp` and `flash` show in the studio's preview and change at once on the stream, so don't promise a viewer a rolling score.
- A Ticker moves on its own: `speed` in px per second (80–160 on air, 0 holds it still); its items scroll in a loop.
- Rows of a repeat each enter on their own; a frame can stagger its layers' entrances (the operator sets it in the frame's inspector).
- Frame transitions: how the whole frame comes on air is its own `transition` field, next to `name` and `code` (`scene_set('frame/<id>', { edits: { transition: { kind: "fade", ms: 450 } } })`) — cut · fade · slide · wipe · dip. A take from the live panel or the script can override it for one cut.

Timing on air: straps and badges 300–500 ms, bars and wipes 400–700 ms, exits shorter than entrances (150–250 ms). Keep motion for what changes; a whole frame of movement reads as noise.

Check a moment of it with `scene_get('frame/<id>', { as: 'image', at: <ms since on air> })`.

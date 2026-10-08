---
name: scripting-the-show
description: "Automates a Streamloop scene with its script through the Streamloop MCP — schedules, reactions to data, source status, frame changes and the operator's buttons — with handlers that change state or take frames to air, and declares controls (toggle, text, number, select, button) and state fields. Use for \"every N minutes…\", \"when the feed goes live…\", \"add a button / toggle\", \"keep a score\" on a Streamloop scene."
---
# Scripting the show

> With the Streamloop MCP: every `scene_*` tool also takes `scene`, the scene's id (`scn_…`) — a call written `scene_get('frame/intro', { as: 'image' })` is `scene_get { scene, path: "frame/intro", as: "image" }`. Changing or removing something that exists also takes its `revision` (a layer: its own or its frame's; `"*"` overwrites on purpose), or the answer is REVISION_REQUIRED. Every set changes the draft only: viewers see it after `publish_scene { scene, draftRevision }` (the `draftRevision` every answer carries; while a stream plays the scene it answers CONFIRM_REQUIRED — tell the user what changes, then `confirm: true`). Starting one from nothing: skill building-scenes.

Two files: `script/show.ts` (behaviour) and `script/state.ts` (the `State` type). get both before changing either. The script usually has handlers already: add a trigger and a branch to the existing handler; never declare a handler twice. Change them with `edits: [{ old, new }]`. Both are type-checked when set; an error names the line.

## Controls first: most need no script
A control is an operator input declared on the show, not in the script: `scene_set('control/<id>', { value: { type, label, description?, group?, default? } })`. A frame binds its value directly — `{controls.sponsorOn ? <Image … /> : null}`, `value={controls.headline}` — so a toggle that shows a layer, a text the operator types, a number or a choice needs **no handler and no state field**. Write a handler only when the control means something the frame can't bind: a button (an event with no value), or a value that updates state (a counter, a pick that becomes a row).
- `toggle` → boolean; `text` → string (`max` characters); `number` → number (`min`, `max`, `step`); `select` → one of `options` (strings), or with `from: "data.<id>"` a row's id (`optionLabel` shows a field, `allowNone` adds null); `button` → an event, `confirm` asks first.
- The script reads them as `ctx.controls.<id>` (typed; buttons are not there) and gets `onControl({ control, value })` for every change, `{ control }` for a button press.
- A removed control takes its value with it; a type change starts the value over; otherwise the operator's value stays across script edits and restarts.

## Shape
```ts
// controls: scene_set('control/homeGoal', { value: { type: "button", label: "Home scores", group: "Score" } })
export const triggers: Trigger[] = [
  { at: "0 */10 * * * *", name: "headlines" },   // schedules only; the other handlers fire for every event of their kind
]
export function onStart(): Update {
  return { state: { score: { home: 0, away: 0 } } }
}
export function onControl(e: ControlEvent, { state }: Ctx): Update {
  if (e.control === "homeGoal") return { state: [set("score.home", state.score.home + 1)] }
  return {}
}
export function onTick(e: TickEvent): Update {
  if (e.name === "headlines") return { effects: [goFrame("headlines", { transition: "fade" })] }
  return {}
}
```

## Events → handlers
Only schedules are declared (`triggers`); every other handler runs for each event of its kind and branches on the event.
| Declared | Fires | Handler (event) |
|---|---|---|
| (handler present) | the show starts | `onStart(ctx)` |
| `{ at: "<cron>", name }` | on a schedule | `onTick(e: TickEvent)` — `e.name` |
| (handler present) | the operator changes a control or presses a button | `onControl(e: ControlEvent)` — `e.control`, `e.value` (none for a button); branch on `e.control` |
| (handler present) | a table changes | `onData(e: DataEvent)` — `e.source` |
| (handler present) | a feed's status changes: `live`, `waiting` (no encoder yet), `down`, `empty` | `onSource(e: SourceEvent)` — `e.source` (its id), `e.status`; e.g. `if (e.source === "cam") return { effects: [goFrame(e.status === "live" ? "liveFeed" : "wall", { transition: "fade" })] }`. A slate while a feed is down needs no script: it is the source's `fallback` (skill handling-media); script it only when the frame itself should change |
| (handler present) | a frame goes on air | `onFrame(e: FrameEvent)` — `e.frame`, `e.previous` |
| (handler present) | a playlist moves to the next item | `onTrack(e: TrackEvent)` |

Cron has six fields, seconds first: `"0 */10 * * * *"` every 10 minutes, `"0 0 * * * *"` hourly, `"0 30 18 * * 1-5"` 18:30 on weekdays. "Two minutes of news at the top of the hour" is two triggers (`"0 0 * * * *"` to the news frame, `"0 2 * * * *"` back): there is no sleep and no duration on `goFrame`. A script that doesn't exist yet is created with `value: { code }` (`script/state.ts` first when the show gets state, then `script/show.ts`; nothing to get before creating); its types (`Trigger`, `Update`, `SourceEvent`, `goFrame`, …) are ambient, nothing is imported. Every id a handler sees or sends is bare — `e.frame`, `show.frame`, `goFrame("live")` say `live`, not `frame/live`; `e.source` says `cam`, not `rtmp/cam`. A feed whose encoder stops or disconnects goes back to `waiting` (no encoder now); `down` is a fault while it was live. A handler for "the camera is gone" branches on `e.status !== "live"`, not on `down` alone.

Some time after the show starts ("go to the guest 30 s in"): cron is wall-clock, so tick often and measure from `ctx.show.liveSince` — once, by checking the frame on air:
```ts
export const triggers: Trigger[] = [{ at: "*/5 * * * * *", name: "clock" }]
export function onTick(_e: TickEvent, { show }: Ctx): Update {
  const onAir = show.liveSince ? Date.parse(show.now) - Date.parse(show.liveSince) : 0
  return onAir >= 30_000 && show.frame === "headlines" ? { effects: [goFrame("guest", { transition: "fade" })] } : {}
}
```

## What a handler returns
`{ state?, effects? }`, synchronously: no timers, promises or network, and `ctx` is read-only.
- state: a partial State (deep-merged), or ops `set("a.b", v)`, `merge("a", {…})`, `del("a.b")`, `push("list", item)`.
- effects (5 at most): `goFrame(id, { transition: "fade" | "cut" | "slide" | "wipe" | "dip", ms })` (a frame's live inputs, pages and clips start when the frame is cued or taken; `goFrame(id, { prepare: true })` cues it ahead so the take is instant, otherwise the take waits for them, 5 s at most), `adBreak(30 | 60 | 90 | 120)`, `skipTrack(playlistId)`, `log("message")`.
- Helpers: `rotate(list, ctx.meta, "key")` (next item, cycling), `topN(rows, n, "field")`, `changed(e, "field")`, `fmt.time(v, "HH:mm")`.
- Back to where the show was: `onFrame` keeps the frame that was on air before this one in state (`e.previous`, null on the first take: `return e.previous ? { state: [set("prev", e.previous)] } : {}`, a `string` field), and a later handler takes `goFrame(state.prev)` — a frame id held in state is accepted as it is. `e.frame` is the frame just taken, not the one to go back to.

## New state
Add the field to `script/state.ts` first, with a doc comment (it becomes its description), then give it a value in `onStart` — a show that is already running takes the new field's value from there, the rest of its state stays:
```ts
export interface State {
  /** Goals so far, shown on the scoreboard. */
  score: { home: number; away: number }
}
```
A frame reads it as `state.score.home` (skill binding-live-data).

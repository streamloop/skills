---
name: binding-live-data
description: "Shows live values in a Streamloop scene through the Streamloop MCP — state, table rows, a playlist's now playing, controls, the clock and tokens — with bindings, templates, repeats and conditions. Use when a layer on a Streamloop stream should display or depend on data: \"show the score\", \"list the headlines\", \"only when breaking\", \"now playing\", \"the time\". Bringing the data in (a URL, a sheet): shaping-data."
---
# Binding live data

> With the Streamloop MCP: every `scene_*` tool also takes `scene`, the scene's id (`scn_…`) — a call written `scene_get('frame/intro', { as: 'image' })` is `scene_get { scene, path: "frame/intro", as: "image" }`. Changing or removing something that exists also takes its `revision` (a layer: its own or its frame's; `"*"` overwrites on purpose), or the answer is REVISION_REQUIRED. Every set changes the draft only: viewers see it after `publish_scene { scene, draftRevision }` (the `draftRevision` every answer carries; while a stream plays the scene it answers CONFIRM_REQUIRED — tell the user what changes, then `confirm: true`). Starting one from nothing: skill building-scenes.

## What a frame can read
| Root | Holds | Where its shape is |
|---|---|---|
| `state.…` | Show state, written by the script | scene_get('script/state.ts') |
| `data.<id>.rows` | A table's rows (`status` too) | scene_get('source/data/<id>') → its schema's columns |
| `sources.<id>.track.title` / `.next` | A playlist's now playing | — |
| `sources.<id>.status` | A media source's `live`, `waiting`, `down`, `empty`, `idle` (not on a frame on air or cued, so not running) | — |
| `show.frame`, `show.next`, `show.live` | What's on air | — |
| `show.now` | The current time | — |
| `controls.<id>` | The operator's inputs (toggle, text, number, select); declare one with scene_set('control/<id>', { value: { type, label, default?, group? } }) — a toggle reads as a boolean; the picture shows controls at their `default`; no script needed | scene_get('control/*') |
| `tokens.…` | Style tokens | scene_get('tokens') |

Every path is checked when the frame is set: a path that doesn't exist is refused with its name. A value the show doesn't have yet → add it to state (skill scripting-the-show) or a table (skill shaping-data).

## Syntax (in a frame's JSX)
- Binding: `value={controls.guestName}`, `style={{ color: tokens.colors.accent }}`
- Template: ``value={`Now playing: ${sources.music.track.title}`}``
- Condition: `{state.breaking ? <Stack id="alert" …/> : null}`. Else: `{state.live ? <A …/> : <B …/>}`. Several layers under one condition: a fragment, `{cond ? <Video …/> : <><Text …/><Stack …/></>}` — each keeps its x/y (unlike wrapping them in a Stack, which re-flows them).
- Repeat: `{data.stories.rows.map((row) => <Stack id="card" key={row.id} max={6}><Text id="cardTitle" value={row.title} /></Stack>)}` — `max` on the repeated element caps it; part of a list is `slice(data.songs.rows, 1, 4).map(…)` ("the next three").
  - `key` is required: the row's `id` (every row has one — typed-in rows name it, fetched rows get one); `max` caps the count (default 50). Text is always a `<Text value=…/>`, never bare text in a Stack.
  - Put the repeat inside a Stack so the rows lay out; `$index`, `$count`, `$first`, `$last` exist alongside the row (not on it).
  - Rows keep their nesting: `row.meta.price`, `data.quote.rows[0].regularMarketPrice` are fields like any other.
- A crawl takes a list, no repeat: `<Ticker id="crawl" items={data.stories.rows} field="title" …/>` (or `items={slice(data.stories.rows, 0, 5)}`) scrolls each row's `title` in a loop (skill composing-frames, examples). A Ticker has no per-item template or condition and doesn't say which row is passing: a badge per row, or a row shown differently, is a repeat of Stacks, not a Ticker.
- Expressions are these JS operators — `+ - * / %`, `=== !== < <= > >=`, `&& || !`, `? :` — index and member access (`rows[state.index % 6].title`), and only these functions: `date(v, "DD/MM/YYYY")`, `time(v, "HH:mm")`, `upper(v)`, `lower(v)`, `truncate(v, 40)`, `number(v, "0,0.00")` (`0,0` groups thousands, `0` doesn't, `.00` the decimals, text before or after the digits stays — `"$0,0.00"` — a `%` after the digits shows a fraction as a percentage: `number(0.645, "0.0%")` → `64.5%` (a value already in percent: `number(v, "0.0") + "%"`), and a leading `+` always shows the sign: `"+0.00"`; other patterns are refused), `percent(v)`, `fallback(v, "—")` (in a text; a Number's value takes a number: `fallback(v, 0)`), `slice(list, start, end)`, `String(v)`, `Number(v)`, `Boolean(v)`. No other methods (`.map` is only the repeat), loops or calls: filtering, sorting and paging belong in a formula (skill shaping-data) or the script.

## Example: the score from state
```jsx
<Stack id="scoreBug" x={760} y={54} direction="row" gap={24} align="center" padding={[12, 32]} style={{ background: "#0B0F1AE6", borderRadius: 12 }}>
  <Text id="home" value={`HOME ${state.score.home}`} style={{ fontSize: 40, fontWeight: 800, color: "#FFFFFF" }} />
  <Text id="away" value={`${state.score.away} AWAY`} style={{ fontSize: 40, fontWeight: 800, color: "#FFFFFF" }} />
</Stack>
```

---
name: building-scenes
description: Builds a Streamloop scene from nothing to on air with the Streamloop MCP — a live, designed stream (text, graphics, tickers, scoreboards, lower thirds, live data, cameras) instead of a looping video. Use whenever someone wants, for their Streamloop stream, an overlay, a lower third, a score bug, a ticker, a weather or news corner, "graphics on my stream", a layout that changes on its own, or anything that isn't just a video playlist — even if they say "overlay", "graphics" or "show" rather than "scene".
---
# Building scenes

A **scene** (`scn_…`) is what a stream plays when it isn't a video loop. It is made of **frames** — full pictures of layers at `frame/<id>`, one on air at a time — plus the **sources** it reads (tables of data, feeds, cameras, files, playlists), reusable **components**, operator **controls**, and a **script** that reacts to events. You edit its **draft**; viewers see nothing until you publish. Studios that have it open see every edit live, so treat it as shared work.

## The tools, in one breath
- `scene_get` reads anything by path, `scene_set` creates or changes one thing, `scene_remove` deletes, `scene_probe` looks at a URL before it becomes a table. Every one takes `scene`. Answers and errors call them get / set / remove / probe.
- Paths: `""` (what it holds), `element/*` (the building blocks and their props), `frame/<id>`, `frame/<id>/<layer>`, `source/<kind>/<id>`, `component/<id>`, `control/<id>`, `script/show.ts`, `script/state.ts`, `tokens`.
- Around them: `create_scene`, `rename_scene`, `publish_scene`, `discard_scene_draft`, `list_scene_versions`, `revert_scene`, `set_stream_scene`, `get_scene_state`, `control_scene`, `create_scene_ingest_key`, `revoke_scene_ingest_key`, `set_scene_secret`, `delete_scene_secret`, `list_scene_inputs`, and `delete_scene` (permanent; refused while a stream plays it).
- Clips the scene plays in a row (music, a video rotation) come from a playlist it owns: `create_playlist { sceneId, kind, name }` → `apply_playlist_operations` to fill it → `scene_set source/playlist/<id> { value: { playlist: "pl_…" } }`, which a Video or Audio layer then shows or plays.
- All of these are tools of the Streamloop MCP server; a client may show them with a prefix (installed with this plugin, Claude Code lists `mcp__plugin_streamloop_streamloop__scene_get`, …).

## The way through
1. `create_scene { name }` — or `list_scenes` to work in one that exists.
2. Learn the vocabulary once: `scene_get ""`, then `scene_get element/*`. Read `element/<Type>` before you use a type you haven't: props differ (Text wraps only with `width`; a Stack places its children and ignores their x/y).
3. Data first, if any: `scene_probe { url }` shows the rows a table would get — and names the `pick` when the rows are a list inside the document. Then `scene_set source/data/<id>` with `connector`, `url`, `pick`, `refreshSeconds`; the answer says what arrived. A key the user gave goes in with `set_scene_secret` and is named `"$secret:<name>"` — never write a key into a URL or header, and never invent one.
4. A frame: `scene_set frame/<id> { value: { name, description, code } }`, where `code` is one `<Frame background={…}>…</Frame>` element; later layers draw on top of earlier ones. The smallest one that works:
   ```jsx
   <Frame background={{ type: "solid", color: "#0B1B3A" }}>
     <Text id="title" value="Starting soon" x={96} y={440} width={1728} style={{ fontSize: 120, fontWeight: 800, color: "#FFFFFF", textAlign: "center" }} />
   </Frame>
   ```
   Bind live values directly: `value={data.weather.rows[0].temp}`, `{controls.showCard ? <…/> : null}`. A design that repeats is a component (skill writing-components); layout is skill composing-frames. A frame is the whole picture viewers get, with nothing under it: there is no transparent background. An "overlay" on a camera or video is that feed in the same frame, as its background (`{ type: "video", source: "<source id>" }`) or a Video layer under the graphics. A camera or encoder is a source first: `scene_set source/rtmp/<id> { value: { name, fallback? } }` (`scene_get source/rtmp` lists its props; `srt`, `rtsp`, `hls`, `file`, `web`, `playlist` likewise), then `create_scene_ingest_key { scene, sourceId }` gives the user the URLs for OBS or vMix — skill handling-media.
5. Look, every time: `scene_get frame/<id> { as: "image" }` draws it as it goes on air and lists `drawn` and `issues` (outside title-safe, overlapping, text too small, a bound table with no rows). Fix what it says, then stop — one or two rounds, not ten. Every set changes the draft only; viewers see nothing until step 6.
6. `publish_scene { scene, draftRevision }` with the `draftRevision` every `scene_get` and `scene_set` answer carries: exactly the draft you built is published, not someone's half-done edit. `dryRun: true` first if you want the list of changes. If a stream has it on air, the answer is `CONFIRM_REQUIRED`: tell the user what will change on air and ask before sending `confirm: true`.
7. On a stream: `set_stream_scene { streamId, scene }` while the stream is stopped, then `start_stream` — it puts the scene in front of the public, so say so and get a yes first. It opens on the first frame `scene_get frame/*` lists (the one made first); the script or the operator takes the others to air (skill operating-live-scenes).

## Why the answers look the way they do
- Every write is built and checked exactly as the studio checks it, and a refusal changes nothing. Read `errors` — each problem with its line or path; one that says "already in the draft" is not yours but must be fixed in the same set.
- Changing or removing something that exists takes the `revision` you read it at (a layer also takes its `frame/<id>`'s; a layer's answer hands you the frame's new `frameRevision` for the next one). Without one the call is `REVISION_REQUIRED`; `"*"` replaces whatever is there, knowingly. A `CONFLICT` means someone else edited it meanwhile: read again, don't overwrite. Creating a new path needs none.
- Code is stored formatted (props alphabetical, one per line). For a text edit copy `old` from what `scene_get` answered; `EDIT_NO_MATCH` quotes the closest line.
- Positions are design pixels (1920×1080 unless the scene says otherwise) from the parent's top-left.

## When something goes wrong
| Code | What it means | Do |
|---|---|---|
| `CHECK_FAILED` / `SCENE_INVALID` | the change or the whole scene would break | fix each entry of `errors`, send again |
| `CONFLICT` | it moved since your revision | `scene_get` it, redo the change on what is there |
| `EDIT_NO_MATCH` / `EDIT_AMBIGUOUS` | `old` isn't there / is there twice | copy from the stored code; add context |
| `IN_USE` | something still uses it | remove or change the users first (or in the same call) |
| `CONFIRM_REQUIRED` | the scene is on air, or the step can't be undone (`delete_scene`) | say what will happen, ask the user, then `confirm: true` |
| `STALE_PUBLISH` | someone published since your draft was based | tell the user; rebuild on the current draft (skill operating-live-scenes) |
| `NOT_FOUND` | wrong id or path | list them: `scene_get frame/*`, `list_scenes` |

## More
- Layout, colour, type and broadcast conventions: skill composing-frames. Motion: animating-frames.
- Tables, formulas, bindings: shaping-data, binding-live-data. Media, cameras, encoders: handling-media.
- Components: writing-components. Script, timers, controls: scripting-the-show. Web pages as layers: coding-web-pages.
- Running it on air: operating-live-scenes. Streams themselves: running-24-7-loops, streaming-to-destinations.

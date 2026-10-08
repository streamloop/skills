---
name: operating-live-scenes
description: Runs a Streamloop scene while it is on air with the Streamloop MCP — taking frames to air with transitions, cueing the next one, setting the operator's controls (a guest's name, a score), replacing a table's rows, skipping a playlist track, changing what's published without going off air, and giving an encoder (OBS, vMix) its stream key. Use for "go to the guest shot", "put up the lower third", "update the score", "change the headline now", "what's on air?", "where does OBS send?", or any change to a scene that viewers are watching.
---
# Operating live scenes

On air, every call here is seen by viewers at once. Say what you are about to do, and for anything the user didn't ask for in so many words, ask first.

## What is on air
`get_scene_state { scene }` — each stream that plays it: the stream's state, and while it runs the frame on air (`frame: "<id>"`), the one cued next (`next`), the controls' values, each source's status (live / waiting / down) and what its playlists play. Read it before acting: it is how you know which `frame/<id>` ids and controls exist on air (the draft may differ from what was published).

## Acting
`control_scene { scene, op, … }` (`streamId` too when more than one stream plays it):
- `goFrame { target, transition? }` — take `frame/<target>` to air; `transition: { kind: "cut" | "fade" | "slide" | "wipe" | "dip", ms? }` (default: the frame's own).
- `next { target }` — cue `frame/<target>` so the take is instant (its inputs start now); `next` with no target clears the cue.
- `setControl { controlId, value }` — the operator's inputs: a toggle, a text, a number, a choice; a button with no value fires it. A layer bound to `controls.<id>` changes at once — no publish needed.
- `setData { sourceId, rows }` — replace a table's rows (a manual table, a quick correction).
- `skip { sourceId }` — a playlist source to its next item (name the source: `get_scene_state` lists the playlists it plays).
Actions are checked against the published scene first (a frame or control it doesn't have is refused with `INVALID_INPUT`). `SCENE_OFFLINE` means no stream is playing it: `set_stream_scene` (stream stopped) then `start_stream`. A `SCENE_TIMEOUT` means the action may or may not have applied: send an `actionId` (a fresh uuid per action) with every action, then a retry with the same id replays instead of acting twice; without one, never retry a `skip` or a button blind — read `get_scene_state` first. A publish keeps the frame on air (when the new version still has it) and the controls' values; the layout changes under them.

## Changing the scene itself while it runs
Edit the draft as always (`scene_get` the resource for its code and `revision`, `scene_set` it with `edits: [{ old, new }]`; skill building-scenes), then `publish_scene` with the `draftRevision` the last `scene_set` or `scene_get` answered: the running stream switches to the new version without going off air. Because viewers see it, publish answers `CONFIRM_REQUIRED` naming the streams — describe the change (`dryRun: true` lists it) and ask before `confirm: true`. `STALE_PUBLISH` means someone published since your draft was based: publishing yours would take their changes off air, and the answer lists them. Tell the user; rebuild on the current draft, or send `overwritePublished: <the version it names>` only if they agree. When the version on air is a revert (`publishedVersionIsRevert`), reading again changes nothing: the choice is `overwritePublished` or `discard_scene_draft`. A mistake on air: `revert_scene { version, expectedVersion, confirm: true }` (`expectedVersion`: the latest published version you saw, from `list_scene_versions`) puts an earlier version back at once; the draft keeps your edits (`discard_scene_draft { draftRevision }` makes it match — `dryRun: true` first shows what would be lost).

## Encoders and keys
An `rtmp` or `srt` source receives a feed from OBS, vMix or a hardware encoder. `create_scene_ingest_key { scene, sourceId }` answers the RTMP, RTMPS and SRT addresses with the key — shown once, so hand them to the user straight away. The source's status is `waiting` until an encoder connects with that key, and `live` while it publishes. `list_scene_inputs` shows the keys (never the key itself), whether each is live, the scene's secrets and cameras; `revoke_scene_ingest_key { scene, keyId, confirm: true }` (the `ingkey_…` from `list_scene_inputs`) cuts an encoder off for good. A source that drops shows its fallback (a slate) until the feed is back.

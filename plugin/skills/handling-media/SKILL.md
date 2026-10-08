---
name: handling-media
description: "Sets up the media sources of a Streamloop scene through the Streamloop MCP — RTMP/SRT feeds, guest and IP cameras (RTSP), files, HLS, web pages by URL (writing one: coding-web-pages), playlists — with their status, fallbacks when a feed drops, and sound levels. Use for \"add a camera/feed\", \"show the guest\", \"what if the feed drops\", \"play this video in the scene\", \"too loud\" on a Streamloop scene."
---
# Handling media

> With the Streamloop MCP: every `scene_*` tool also takes `scene`, the scene's id (`scn_…`) — a call written `scene_get('frame/intro', { as: 'image' })` is `scene_get { scene, path: "frame/intro", as: "image" }`. Changing or removing something that exists also takes its `revision` (a layer: its own or its frame's; `"*"` overwrites on purpose), or the answer is REVISION_REQUIRED. Every set changes the draft only: viewers see it after `publish_scene { scene, draftRevision }` (the `draftRevision` every answer carries). Starting one from nothing: skill building-scenes.

Media sources are `source/<kind>/<id>`: rtmp, srt, camera (a guest joins by link), rtsp (an IP camera the platform pulls), file, hls, web, playlist. scene_get('source/<kind>') lists a kind's props; scene_get('source/*') lists its sources.

## Showing one
- `<Video id="cam" source="studioFeed" w={1920} h={1080} fit="cover" />`, or as the frame's background: `background={{ type: "video", source: "studioFeed" }}`.
- A feed that may be off: `<FirstAvailable id="main" sources={["studioFeed", "guestCam"]} w={1920} h={1080} />` shows the first one that is live.
- A place the operator fills by hand: a `Video` with no source, `<Video id="slot" w={960} h={540} />`.
- A source's sound without its picture: `<Audio id="bed" source="music" duck={40} />` at the bottom of the frame (background audio).
- An audio-only source (`audioOnly: true`: an audio playlist, a music file) plays only that way: never in a Video, a FirstAvailable or a video background (the check refuses it).

## Off air and back
A file or playlist pauses while no on-air frame plays it; a frame change that keeps it on air never stops it. Its `resume` says what it plays when it comes back: `"continue"` (default, where it stopped), `"next"` (the next item) or `"restart"` (from the first item, at its start; a single file: from its start). Set it on the source (`edits: { resume: "restart" }`): every frame showing it follows it. Live feeds have none.

## A music bed
A file source (one audio file: `scene_set('source/file/bed', { value: { name, description, src, loop: true, volume: 15 } })`) plus `<Audio id="bed" source="bed" duck={40} />` in every frame that plays it. The same source in two frames keeps playing across the cut — no restart, nothing to script. `duck` is automatic: the bed is lowered by that % whenever a voice on a camera source in the frame speaks (voice detection); a literal number, no control, state or binding.

## When a feed drops
A source's `fallback` decides what its layers show after `delay` seconds without signal: `{ mode: "slate", text: "Back shortly", delay: 3 }`, `{ mode: "source", sourceId: "backupFeed", delay: 3 }`, or `{ mode: "none" }` (hide). Set it with `edits: { fallback: { … } }` on the source. Its state is at `source/<kind>/<id>/status` (live · waiting · down, with the reason; idle while no frame on air or cued names it: sources run on demand, and a take waits for them unless the frame was cued ahead); frames can read `sources.<id>.status`.

## Sound
`volume` in percent (100 = as it arrives, the default; 0 = silent; up to 200) and `muted`, on the source.

## IP cameras
An `rtsp` source is a camera on the internet (its RTSP URL; port 554 forwarded). Its address and password are typed by the operator in the Streamloop studio (the source's dialog) and kept on the server: you never see or set them, so create the source (`{ name, fallback }`), place it, and tell the operator to open it in the studio, paste the camera's URL and press Test connection. A camera that can push RTMP itself is better as an `rtmp` source. Its audio: AAC or G.711 plays; anything else is video only (the status says so).

## Encoders
An rtmp/srt source's stream key comes from `create_scene_ingest_key({ scene, sourceId })`: RTMP, RTMPS and SRT URLs with the key, in that answer only — hand them to the user for OBS or vMix.

---
name: running-24-7-loops
description: Sets up and runs 24/7 video loops on Streamloop with its MCP — uploading or importing videos and music, filling and ordering playlists, scheduling (once, daily, weekly) and starting, checking why a stream won't go live, and changing a live stream's playlist without going off air. Use whenever someone wants a lofi channel, an always-on stream, "loop these videos to YouTube", "add this clip to my stream", "stream every weekday at 9", a custom play order, or asks why their stream isn't starting.
---
# Running 24/7 loops

A **stream** (`stream_…`) loops its **video playlist** (with an optional **audio playlist** under it) to its **destinations**, now or on a schedule, on Streamloop's servers. It spends the workspace's credits while it runs. Everything belongs to one **workspace**: an empty list is more often the wrong workspace than an empty account (`get_selected_workspace`, `list_workspaces`, `select_workspace`).

## Setting one up — the order matters
1. `create_stream { name, quality, framerate }`. It comes with two empty playlists, one video and one audio.
2. Media in, one of two ways:
   - **A link** (a video page, a direct file, a file the user attached in the chat): `discover_external_media { url }` → `import_upload_from_url { url, selections }` with the asset and quality ids it handed out (they can't be made up), then `get_upload` until it's finished.
   - **A local file**: `create_upload` → PUT the bytes to `uploadUrl` with the same Content-Type (do it yourself if you can make HTTP requests, else hand the URL to the user) → `complete_upload`. Up to 1.5 GB per file (5 GB on a paid plan): split longer videos.
3. `list_playlists { streamId }` → the video playlist; `apply_playlist_operations` to add the media (all-or-nothing; pass `basedOnVersion` from `get_playlist` so a concurrent edit is reported, not overwritten).
4. A destination: skill streaming-to-destinations, then `add_stream_destination`.
5. `check_stream_readiness` — every blocking issue at once — then `start_stream` or `schedule_stream`. There is nothing to publish before a first start: it takes the playlist as it is. Read `state` in `start_stream`'s answer: `invalid` means refused (`started: false`, with `issues` — fix them and start again); `preparing` means accepted, live within about a minute. A just-completed upload's media facts arrive about 15–25 s later: wait that long before starting on it.

## Schedules (the rules that bite)
- Every time carries an explicit UTC offset: `2026-09-22T21:00:00-04:00`, never a bare local time. The user's clock is not UTC.
- `repeat: daily | weekly` needs an end time and a window under 24 h; weekly `days` are the UTC weekdays of the start instant — a Monday 9 pm Eastern window repeats on Tuesday in UTC. The answer echoes the window in UTC: read it back to the user.
- A schedule only arms a window: the stream must already be ready (`check_stream_readiness`).

## Changing a live stream
Edit its playlist as above, then `swap_live_playlist { playlistId, expectedDraftVersion }` — the one place an edit isn't enough on its own: a live stream keeps playing what it started with until the swap. It is asynchronous (new media may need encoding): poll `get_playlist`. On a stream that isn't live it fails with `NOT_LIVE` and isn't needed.

## Play order
`create_playlist_order { playlistId, name, intent }` turns plain language in `intent` ("never two from the same artist in a row, a jingle every 5 songs") into an order — always with the `playlistId`, or the rule is written against no items. Its `status`: `READY`; `INVALID` (read `diagnostics`, then `refine_playlist_order { id, message }`); `MESSAGE` (a question back — ask the user); `PENDING` (poll `get_playlist_order`). Creating doesn't apply it: `assign_playlist_order { playlistId, orderId }` does; `unassign_playlist_order` goes back to the list order; `list_playlist_orders` shows the orders already written.

## Finding, stopping, reusing
- `list_streams`, `get_stream` — what exists and its state. `list_uploads` — media already in the workspace (reuse it before uploading again).
- `stop_stream` ends a live or starting stream; nothing is deleted. A daily or weekly stream stopped this way comes back at its next window: `cancel_stream_schedule` too, to keep it off.
- `copy_playlist` reuses one playlist's contents on another stream (the target must be empty).

## When it won't start
`check_stream_readiness` names configuration problems (no destination or an archived one, an empty playlist, media too big for the stream's storage, a scene missing or not published) — not credits, not whether an upload has finished processing. Ready but still not live: `get_billing_overview` — credits are checked at the moment it activates, and buying credits happens in the Streamloop dashboard, not here. Errors carry a code in brackets (`[STREAM_LIVE]`, `[IN_USE]`): act on the code.

## Safety
`start_stream`, `schedule_stream` and `swap_live_playlist` put video in front of the public; `delete_stream` and `delete_upload` are permanent and take `confirm: true`. Say what will happen and get a yes first.

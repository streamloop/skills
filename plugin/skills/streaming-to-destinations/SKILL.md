---
name: streaming-to-destinations
description: Connects Streamloop streams to where they're watched with its MCP — YouTube (a managed channel connection, or a stream key), Twitch, Facebook, Kick, X, TikTok, LinkedIn, Telegram or any RTMP server — and multistreams one stream to several at once, turning one off while live. Use whenever someone with a Streamloop stream says "put it on YouTube", "also send it to Twitch", "stream to two platforms", "my YouTube stopped", "change the title or thumbnail", or pastes a stream key for it.
---
# Streaming to destinations

A **destination** (`dest_…`) is where a stream's output goes. A stream publishes to up to 5 at once (multistream, where the workspace has it), at most one of them YouTube, all from one encode.

## Making one — never guess an address
Start with `get_connection_hints { platform }`: it gives the exact ingest URL (a wrong host is a stream that silently never connects), where the user finds their key in the platform's UI, the tools to call in order, and what ends a 24/7 stream early on that platform.
- **YouTube, managed** (titles, privacy, thumbnail, viewer counts from here): `start_youtube_connection` → give the user the Google consent URL (an agent can't consent) → `list_youtube_channels` → `create_youtube_destination { channelId }` with the id from that list, not a channel URL or handle. Also the fix for a destination that says `needs_reauth` or `revoked`.
- **Any RTMP key** (Twitch, Facebook, Kick, X, TikTok, Telegram, YouTube by key, a custom server): `create_rtmp_destination { name, rtmpUrl, rtmpKey }` with `rtmpUrl` from the hints. The key is stored encrypted and never read back. A YouTube key streams video only: nothing about the broadcast can be managed through it.

## Putting it on a stream
- `add_stream_destination { streamId, destinationId }` — refused while the stream is live (`STREAM_LIVE`), beyond 5 or a second YouTube (`DESTINATION_LIMIT`, `YOUTUBE_DESTINATION_LIMIT`), without multistream for a second one (`MULTISTREAM_NOT_ENABLED`), or when the stream's quality or codec can't reach it (`DESTINATION_INCOMPATIBLE`, with the reason).
- While live you can only stop sending to one: `set_stream_destination_enabled { enabled: false }` takes that output off the air at once and the others stay up. Not the last one (`LAST_DESTINATION`: stop the stream instead). Turning one back on, adding or removing waits until the stream is stopped.
- `list_stream_destinations { streamId }` shows each output's state and why it's off.
- `remove_stream_destination` takes one off a stopped stream; the destination stays in the workspace.

## Keeping destinations working
`list_destinations` shows every destination of the workspace and its `status`: only `active` can carry a stream; `needs_reauth` or `revoked` YouTube connections need `start_youtube_connection` again. `update_destination` renames one, replaces a custom RTMP key the platform rotated (a live stream keeps the old key until it restarts), or archives it out of the picker (reversible).

## YouTube broadcast settings
With a managed YouTube destination, `update_stream { id, youtube: { title, description, privacy, … } }` sets the broadcast; `get_stream_live_stats` reads viewers. Applied settings stay even if the YouTube part fails — read `youtubeSettingsError`. After a reconnect, `retryMirror: true` mirrors the schedule onto YouTube again.

## Costs, briefly
Each destination after the first adds a flat amount per hour of streaming (it depends on the stream's quality), and YouTube's backup ingest (`runBackupStream`) doubles the streaming cost. `get_billing_overview` shows the rate card; nothing here can spend or buy credits.

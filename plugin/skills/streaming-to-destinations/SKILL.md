---
name: streaming-to-destinations
description: Connects Streamloop streams to where they're watched with its MCP — YouTube (a managed channel connection, or a stream key), Twitch, Facebook, Kick, X, TikTok, LinkedIn, Telegram or any RTMP server — and multistreams one stream to several at once, turning one of several off while live. Use whenever someone with a Streamloop stream says "put it on YouTube", "also send it to Twitch", "stream to two platforms", "my YouTube stopped", "change the title or thumbnail", or pastes a stream key for it.
---
# Streaming to destinations

A **destination** (`dest_…`) is where a stream's output goes. A stream publishes to up to 5 at once (multistream: a workspace feature, else the second add is `MULTISTREAM_NOT_ENABLED` and the person turns it on in the dashboard), at most one of them YouTube, all from one encode. The stream itself: `list_streams { name }` finds it and its state; `list_stream_destinations { streamId }` what it sends to.

## Making one — never guess an address
Start with `get_connection_hints { platform }` (`youtube`, `twitch`, `facebook`, `x`, `kick`, `tiktok`, `linkedin`, `telegram`, or `other`): it gives the exact ingest URL (a wrong host is a stream that silently never connects; `null` when the platform issues one per session — then ask the user for the URL their platform shows next to the key), where the user finds their key in the platform's UI, the tools to call in order, and what ends a 24/7 stream early on that platform.
- **YouTube, managed** (titles, privacy, thumbnail, viewer counts from here): `start_youtube_connection` → give the user the Google consent URL (an agent can't consent) → `list_youtube_channels` → `create_youtube_destination { channelId }` with the id from that list, not a channel URL or handle. Also the fix for a destination that says `needs_reauth` or `revoked`.
- **Any RTMP key** (Twitch, Facebook, Kick, X, TikTok, Telegram, YouTube by key, a custom server): `create_rtmp_destination { name, rtmpUrl, rtmpKey }` with `rtmpUrl` from the hints. The key is stored encrypted and never read back. A YouTube key streams video only: nothing about the broadcast can be managed through it.

## Putting it on a stream
- `add_stream_destination { streamId, destinationId }` — refused while the stream is live (`STREAM_LIVE`), beyond 5 or a second YouTube (`DESTINATION_LIMIT`, `YOUTUBE_DESTINATION_LIMIT`), without multistream for a second one (`MULTISTREAM_NOT_ENABLED`), or when the stream's quality or codec can't reach it (`DESTINATION_INCOMPATIBLE`, with the reason).
- While live the only change is stopping one of several: `set_stream_destination_enabled { streamId, destinationId, enabled: false }` takes that output off the air at once and the others stay up. Not the last one (`LAST_DESTINATION`: stop the stream instead). Turning one back on, adding or removing waits until the stream is stopped: `stop_stream`, the changes, `start_stream` — about a minute off air, so say so and get a yes first. Nothing here is timed: "YouTube off for an hour" is a disable now and, in an hour, a stop, enable and start.
- `list_stream_destinations { streamId }` shows each output's state and why it's off.
- `remove_stream_destination` takes one off a stopped stream; the destination stays in the workspace.

## Keeping destinations working
`list_destinations` shows every destination of the workspace and its `status`: only `active` can carry a stream; `needs_reauth` or `revoked` YouTube connections need `start_youtube_connection` again (the user opens the consent URL; then read `list_destinations` — the same destination is `active` again, and a stream using it can start. Only if it isn't: `list_youtube_channels` → `create_youtube_destination` and attach the new one). `update_destination` renames one, replaces a custom RTMP key the platform rotated (a live stream keeps the old key until it restarts), or archives it out of the picker (reversible).

## YouTube broadcast settings
With a managed YouTube destination, `update_stream { id, youtube: { title, description, privacy, … } }` sets the broadcast; `get_stream_live_stats` reads viewers. Applied settings stay even if the YouTube part fails — read `youtubeSettingsError`. After a reconnect, `retryMirror: true` mirrors the schedule onto YouTube again.

## Costs, briefly
Each destination after the first adds a flat fee per 30 days by resolution ($3.50 at 720p, $6 at 1080p, $8 at 1440p, $12.50 at 4K), and YouTube's backup ingest (`runBackupStream`) doubles the streaming cost. `get_billing_overview` shows the balance; an extra destination spends credits once the stream runs, and nothing here can buy them.

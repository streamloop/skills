---
name: streamloop
description: What Streamloop is and how a stream gets on air — the two kinds of loop (pre-recorded video, scene), what to choose for a given channel, the way from nothing to live, quality and cost, and whether to work through the Streamloop MCP, its HTTP APIs or the dashboard. Read first whenever Streamloop comes up and the task isn't yet one of the specific skills: "stream my videos 24/7", "what do I need to go live", "which quality", "how much does it cost", "can it show my scoreboard", "should I use a scene", "how do I automate this".
---
# Streamloop

Streamloop runs a live stream from the cloud, around the clock: no OBS, no encoder, no PC left on. A **loop** is a stream set up once and left running; the API and the MCP call it a **stream** (`stream_…`). It plays to **destinations** (YouTube, Twitch, Kick, Facebook, X, TikTok, LinkedIn, Telegram or any RTMP server; up to 5 at once from one encode, at most one YouTube) and spends the workspace's **credits** while live (plus a one-time encoding fee per uploaded file). Streamloop watches every stream and restarts it if it stalls (over 99.9 % uptime); YouTube and RTMP keys are encrypted at rest.

## Two kinds of loop: choose first

| | Pre-recorded video | Scene |
| --- | --- | --- |
| What plays | uploaded videos on repeat, with an optional audio playlist over them | a designed live show: frames of layers, cut between |
| Quality | up to 4K 60 fps, HDR | up to 1080p 60 fps |
| Playlists, background music, smart order | yes | yes |
| Live inputs (RTMP, SRT, RTSP cameras, HLS) | no | yes |
| Web pages, text, logos, clocks, live data (sheets, CSV, JSON) | no | yes |
| Overlays, lower thirds, tickers, transitions, a fallback when a feed drops | no | yes |
| A script that reacts to time and data; operator controls | no | yes |
| Availability | everyone | private beta, by invitation (the workspace must have it) |

Pick **video** when the channel is media on repeat: music, ambience, a show archive, a sales reel. Pick a **scene** when anything has to change on air (scores, headlines, weather), come from a live feed, or sit on top of video. A scene can still play a playlist inside it. Both kinds schedule, multistream, manage a YouTube broadcast, and can run a backup stream.

## The way to live, in order

1. **A workspace**: everything belongs to one; an empty list is usually the wrong workspace, not an empty account.
2. **A stream** with a name, resolution and frame rate — and, for around the clock, `streamDuration: 0`: a run otherwise stops after the account's default run length.
3. **What it plays**: a video playlist of uploads (files or imports from a link), or a scene built and published.
4. **A destination** attached to it: a managed YouTube channel (sign-in, the broadcast is managed), or a stream key for any RTMP platform.
5. **A readiness check**, then **start** now or on a **schedule** (once, daily, weekly; every time with its UTC offset). Live within about a minute. While live, an edited playlist goes on air with one more call (a swap) and an edited scene with a publish; the stream doesn't stop.

A second destination on one stream (multistream) is a feature the workspace must have; without it the add is refused with `MULTISTREAM_NOT_ENABLED`, and the person turns it on in the dashboard.

## Quality and cost

Resolution 720p, 1080p, 1440p or 4K (scenes to 1080p); frame rate 24, 25, 30 or 60. **1080p at 30 fps suits most channels.** Credits: $1 is 1,000,000. Per hour live, 1080p 30 fps is about $0.0139 (about $10 a month around the clock; 720p about $5, 4K about $30); 60 fps and higher resolutions cost more. On top: encoding each uploaded file once, $0.005 per minute of media; each destination after the first, a flat fee per 30 days by resolution ($3.50 at 720p, $6 at 1080p, $8 at 1440p, $12.50 at 4K); YouTube's backup ingest doubles the streaming cost; a scene adds rendering (a 1080p 30 fps scene loop is about $20 per 30 days: $10 streaming, $10 rendering, at the beta price). `get_billing_overview` answers the rate card and the balance. Files up to 1.5 GB each until the account has bought credits, 5 GB after. A new account can claim $5 of credits once (`get_billing_overview` says whether it still can, `eligibleForFreeCredits`); nothing an agent does can buy credits.

## Which way in

- **The Streamloop MCP** (`https://mcp.streamloop.app/mcp`, OAuth: the user signs in as themself and approves scopes) is the way for an assistant acting for someone: what the dashboard does with loops, media, destinations and scenes is a tool, answers say what comes next, refusals say why. Signing in, paying, YouTube consent, turning multistream on and workspace membership stay with the person in the dashboard. The skills below assume it.
- **The HTTP APIs** (`https://api.streamloop.app/v1`, REST; `/v1/scenes`, the Scenes API; an API key in `X-API-Key`) are for code: scripts, backends, cron, CI, no-code flows. Same objects, same error codes. Skill: calling-the-streamloop-api.
- **The dashboard** (`streamloop.app/loops`) and, for scenes, the **studio** with its own assistant, are the user's hands. Point a person there for sign-in, payment, YouTube consent and anything an agent can't do for them.

## Then read

| The task | Skill |
| --- | --- |
| uploads, playlists, schedules, a stream that won't start, a live playlist swap | running-24-7-loops |
| YouTube, Twitch, any RTMP; multistream; one destination off while live | streaming-to-destinations |
| which workspace, credits left, burn rate, past usage | managing-workspaces-and-credits |
| a scene from nothing to on air | building-scenes, then composing-frames, animating-frames, shaping-data, binding-live-data, scripting-the-show, handling-media |
| a design to reuse across frames (a lower third, a score bug, a card) | writing-components: the show's own elements composed into one |
| a layer the show's elements can't draw (rich CSS, SVG, charts, canvas), or the user asks for HTML or React | coding-web-pages: a web page rendered in a layer |
| a scene while it is on air | operating-live-scenes |
| a script or backend without the MCP | calling-the-streamloop-api |

## Words that differ by surface

Loop (dashboard) = stream (API, MCP). Scene (`scn_…`) is what is built and published; a **frame** (`frame/<id>`) is one arrangement of its layers and the one thing on air at a time. A **destination** (`dest_…`) is where output goes. A **workspace** holds streams, uploads, destinations and credits, shared by its members.

# Streamloop skills

Skills that teach an AI agent to run [Streamloop](https://streamloop.app): 24/7 video loops, streaming to YouTube, Twitch and any RTMP platform (one stream to up to 5 at once), and live scenes, graphics, data and cameras built and taken to air. They work with the Streamloop MCP server at `https://mcp.streamloop.app/mcp` (OAuth, sign in as yourself) and, for code, the REST and Scenes APIs.

## Install

Any agent that reads skills:

```bash
npx skills add streamloop/skills
```

Claude Code, which also adds the MCP server:

```text
/plugin marketplace add streamloop/skills
/plugin install streamloop@streamloop
```

Only the MCP server, no skills: `claude mcp add --transport http streamloop https://mcp.streamloop.app/mcp`, or add `https://mcp.streamloop.app/mcp` as a connector in Claude, ChatGPT or Cursor.

## Skills

| Skill | For |
|---|---|
| `running-24-7-loops` | streams that loop videos and music: uploads, playlists, schedules, live playlist swaps, play orders |
| `streaming-to-destinations` | YouTube (managed or by key), Twitch and any RTMP platform, multistream |
| `managing-workspaces-and-credits` | the right workspace, credit balance and burn rate, past usage |
| `calling-the-streamloop-api` | scripts and backends on the REST and Scenes HTTP APIs, without the MCP |
| `building-scenes` | a live scene from nothing to on air: the scene tools, the way through, what answers mean |
| `composing-frames`, `animating-frames` | layout, type, colour and motion of each frame (`frame/<id>`) |
| `shaping-data`, `binding-live-data` | tables from URLs, sheets and formulas, and binding them in layers |
| `writing-components` | reusable lower thirds, score bugs, cards |
| `scripting-the-show` | timers, reactions to data and feeds, controls, state |
| `handling-media` | cameras, encoders, files, web pages, fallbacks, sound |
| `coding-web-pages` | a web page written as a layer |
| `operating-live-scenes` | taking frames to air, controls, live corrections, encoder keys |

Each skill is `plugin/skills/<name>/SKILL.md`.

## Where they come from

The scene skills are the same ones Streamloop's studio assistant reads, generated from the studio's source (`studio/src/ai/skills` in `streamloop/live-scene`); the other four are written for agents outside the studio. A release of the studio copies `plugin/streamloop` there into `plugin/` here. Issues and suggestions are welcome here; the text itself is fixed at the source.

Docs: https://streamloop.app/docs · MCP reference: https://streamloop.app/docs/api-reference/mcp/overview

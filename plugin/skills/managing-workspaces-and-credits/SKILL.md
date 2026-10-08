---
name: managing-workspaces-and-credits
description: Answers "which account, which workspace, how many credits, how much did I stream" on Streamloop with its MCP — finding things that seem missing (usually another workspace is selected), switching workspaces, reading the credit balance and burn rate, estimating how long streams can keep running, and reporting past streaming usage. Use whenever a Streamloop list comes back empty or something the user named isn't found there, when a ready stream won't go live, or for Streamloop's "how many credits do I have", "how long can this run", "what did I spend last month", "switch to my team's workspace".
---
# Workspaces and credits

Everything — streams, scenes, uploads, playlists, destinations, credits — belongs to one **workspace** (`wksp_…`), and nothing is shared between workspaces. A person can belong to several (a personal one and a team's). So when something the user is sure exists isn't in a list, the likely answer is the wrong workspace, not a missing thing: check before you say it doesn't exist.

## Finding the right workspace
- `get_selected_workspace` — which one this connection uses now, and why (`explicit`, `header`/`env`, `token-hint`).
- `list_workspaces` — every workspace with the user's role, and which is selected.
- `select_workspace { workspaceId }` — switches this connection only (other chats and clients are unaffected). Say which one you switched to.
- `get_account` — who the login belongs to; it says nothing about workspaces or credits.

Inviting members, roles and API keys are managed in the Streamloop dashboard, not through the MCP.

## Credits
Streaming spends the workspace's credits while a stream runs ($1 buys 1,000,000 credits); the rate depends on output quality, and each extra multistream destination adds to it.
- `get_billing_overview` — balance, credits left, good standing, what is being spent right now, which streams are live and their cost, and the per-minute rate per quality. Estimate remaining time from the balance and the current burn rate; say it's an estimate.
- `get_streaming_usage { period: DAY | WEEK | MONTH | QUARTER | YEAR, startDate?, endDate? }` — minutes streamed and credits spent in the past, by quality. Use RFC3339 times with an offset.

`check_stream_readiness` doesn't look at credits; they are checked when a stream activates. So a stream that is ready but won't go live (or stopped by itself) is a reason to read `get_billing_overview`.

Credits can't be bought from here: if the balance is short, send the user to the Streamloop dashboard. Don't promise prices or plan limits you haven't read from a tool's answer or description.

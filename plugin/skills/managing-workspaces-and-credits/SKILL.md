---
name: managing-workspaces-and-credits
description: Answers "which account, which workspace, how many credits, how much did I stream" on Streamloop with its MCP — finding things that seem missing (usually another workspace is selected), switching workspaces, reading the credit balance and burn rate, estimating how long streams can keep running, and reporting past streaming usage. Use whenever a Streamloop list comes back empty or something the user named isn't found there, when a ready stream won't go live, or for Streamloop's "how many credits do I have", "how long can this run", "what did I spend last month", "switch to my team's workspace".
---
# Workspaces and credits

Everything — streams, scenes, uploads, playlists, destinations, credits — belongs to one **workspace** (`wksp_…`), and nothing is shared between workspaces. A person can belong to several (a personal one and a team's). So when something the user is sure exists isn't in a list, the likely answer is the wrong workspace, not a missing thing: check before you say it doesn't exist.

## Finding the right workspace
- `get_selected_workspace` — which one this connection uses now, and why (`explicit`, `header`/`env`, `token-hint`).
- `list_workspaces` — every workspace with the user's role, and which is selected.
- `select_workspace { workspaceId }` — switches this connection only (other chats and clients are unaffected). Say which one you switched to. From then on every list and every credit figure is that workspace's: `list_streams`, `list_uploads`, `list_destinations`, `list_scenes` are the lists to check, and `get_billing_overview` reads its balance, not the one you left.
- `get_account` — who the login belongs to; it says nothing about workspaces or credits.

Inviting members, roles and API keys are managed in the Streamloop dashboard, not through the MCP.

## Credits
Streaming spends the workspace's credits while a stream runs ($1 buys 1,000,000 credits); the rate depends on resolution and frame rate (`q_1080p` at `f_30` is the usual), and each extra multistream destination adds a flat amount per 30 days.
- `get_billing_overview` — balance, credits left, good standing, what is being spent right now, which streams are live and their cost, and the rate card per quality. "How long can I keep streaming": with the streams live, balance ÷ the current burn; with them stopped, balance ÷ (the rate of each stream's quality, added up). Say it's an estimate.
- `get_streaming_usage { period: "MONTH", startDate?, endDate? }` — minutes streamed and credits spent in the past, by quality. `period` is the bucket size (`DAY | WEEK | MONTH | QUARTER | YEAR`); `startDate` and `endDate` are the window, RFC3339 with the user's offset (defaults: a recent span, up to now). "Last month": the first of last month to the first of this month.

`check_stream_readiness` doesn't look at credits; they are checked when a stream activates. So a stream that is ready but won't go live (or stopped by itself) is a reason to read `get_billing_overview`.

Credits can't be bought from here: if the balance is short, send the user to the Streamloop dashboard. Beyond the rate card `get_billing_overview` answers, don't promise prices or plan limits.

---
name: shaping-data
description: "Creates and changes the tables of a Streamloop scene through the Streamloop MCP — typed-in rows, CSV/JSON/XLSX files and URLs, Google Sheets, elements picked from a web page — and formulas that filter, sort, page or join them. Use for \"add a table of…\", \"import this sheet\", \"show the price from this API\", \"only the top 5\", \"three at a time\" on a Streamloop scene."
---
# Shaping data

> With the Streamloop MCP: every `scene_*` tool also takes `scene`, the scene's id (`scn_…`) — `scene_get('frame/intro', { as: 'image' })` below is `scene_get { scene, path: "frame/intro", as: "image" }`. Changing or removing something that exists also takes its `revision` (a layer: its own or its frame's; `"*"` overwrites on purpose), or the answer is REVISION_REQUIRED. Starting one from nothing: skill building-scenes.

A table is what the show reads at `data.<id>.rows` (media sources are at `sources.<id>`). Give every row an `id`. Rows keep their nesting: `row.meta.price` and `data.quote.rows[0].regularMarketPrice` bind and type-check like any column.

## A typed-in table
```
scene_set('source/data/guests', { value: { name: "Guests", description: "Tonight's guests, for the lower thirds", connector: "inline",
  rows: [{ id: "g1", name: "Maria Lopez", title: "Climate correspondent" }, { id: "g2", name: "…", title: "…" }] } })
```
Change rows later with its `revision` and `edits: { rows: [...] }` — a merge patch replaces the whole list.

## From a URL or a web page
1. `scene_probe({ url })` first — nothing changes in the show: the HTTP status, the shape of what came back (keys and values, or the page's title and text) and the rows a table would make of it. Add `connector` and `pick` to see the rows they give; repeat until the rows are right.
2. `scene_set('source/data/<id>', { value: { name, description, connector, url, pick, refreshSeconds } })`. The answer says what arrived — status, rows, columns, the first row — or why nothing did. No need to read status or rows after it.
- `connector`: `json`, `csv` or `xlsx` with `url`; `sheets` with `url` (and `sheet`, the tab); `html` with `url` and `pick`; `file` with `file` (its path in the show folder).
- `sheets`: a sheet shared with anyone with the link just works; a private one is read by the studio's service account, and when Google refuses the error names the account — tell the operator to share the sheet with it (Viewer), then set again.
- `pick` on `json`: a path into the document — an array there is the rows, an object one row: `pick: "chart.result[0].meta"`. Without it the document itself must be an array of rows or one object.
- `pick` on `html`: a CSS selector — each matching element is a row `{ id, text, href?, src? }`.
- `refreshSeconds` re-fetches (5 or more; 0 = once). Fetches go out from Streamloop's servers, as on air: a feed that needs no key works, CORS doesn't matter.
- `headers` sends request headers with every fetch, e.g. `{ Accept: "application/json" }`. A key (Authorization, X-Api-Key, …) is saved as a secret — `set_scene_secret({ scene, name, value })` with a key the user gave you — and the header reads `"$secret:<name>"`. Header values other than secret names read as hidden, and so does a key in the URL's query (`?api_key=(hidden)`): save those with `set_scene_secret` too, and a URL you set keeps them as they are. Probe with the same headers first (a `$secret:` reference works there too).
- Keys are never guessed: no `apikey=demo`, no made-up or borrowed tokens, in a URL or a header — probe and set refuse them. When a feed needs a key, ask the operator for it and stop there. Two failures on one idea: stop, remove the source, ask.

A price card from a public quote feed:
```
scene_probe({ url: "https://query1.finance.yahoo.com/v8/finance/chart/NVDA?interval=1d&range=1d" })   // outline → chart.result[0].meta: { symbol, regularMarketPrice, chartPreviousClose, … }
scene_set('source/data/nvda', { value: { name: "NVDA quote", description: "NVIDIA's price for the market card", connector: "json",
  url: "https://query1.finance.yahoo.com/v8/finance/chart/NVDA?interval=1d&range=1d", pick: "chart.result[0].meta", refreshSeconds: 15 } })
<Text id="price" value={number(data.nvda.rows[0].regularMarketPrice, "0,0.00")} … />
```

## A formula
A pure TypeScript function of declared inputs; it re-runs when one changes. It is also how a fetched table takes a new shape — typed from the inferred columns, nested ones too.
```
scene_set('source/derived/topStories', { value: { name: "Top stories", description: "The first three headlines, for the ticker",
  formula: "export default derive({ from: ['stories'] }, ({ stories }) => stories.rows.slice(0, 3))" } })
scene_set('source/derived/nvdaCard', { value: { name: "NVDA card", description: "Price and change since the previous close",
  formula: "export default derive({ from: ['nvda'] }, ({ nvda }) => { const q = nvda.rows[0]; return [{ id: 'nvda', price: q.regularMarketPrice, change: q.regularMarketPrice - q.chartPreviousClose }] })" } })
```
- Inputs: `from` (table ids), `state`, `controls`, `show`, `clock` (dotted paths). Only declared inputs are visible, as the second argument: `derive({ from: ['stories'], state: ['page'] }, ({ stories }, { state }) => stories.rows.slice(state.page * 3, state.page * 3 + 3))`.
- Return rows, or `{ rows, …fields }`. No network, timers or randomness.
- Frames read it like any table: `data.topStories.rows`.
- Only for a new shape: filtering, sorting, paging, joining, combining. A column the rows already have binds as it is (`data.odds.rows[0].lastTradePrice`, formatted in the binding) — a formula that renames or converts fields is a step the show doesn't need.

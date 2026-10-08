---
name: coding-web-pages
description: "Writes a web page in code (React, Tailwind v4, motion; transparent) as a web source a Streamloop scene shows, with props bound to the scene's data, through the Streamloop MCP. Use when a Streamloop scene needs what its elements can't draw — rich CSS, SVG, charts, complex animation — or the user asks for a web page or HTML inside their Streamloop scene. Not for web pages outside Streamloop."
---
# Coding web pages

> With the Streamloop MCP: every `scene_*` tool also takes `scene`, the scene's id (`scn_…`) — a call written `scene_get('frame/intro', { as: 'image' })` is `scene_get { scene, path: "frame/intro", as: "image" }`. Changing or removing something that exists also takes its `revision` (a layer: its own or its frame's; `"*"` overwrites on purpose), or the answer is REVISION_REQUIRED. Every set changes the draft only: viewers see it after `publish_scene { scene, draftRevision }` (the `draftRevision` every answer carries). Starting one from nothing: skill building-scenes.

```
scene_set('source/web/scoreBug', { value: {
  name: "Score bug", description: "Home and away score; the number pops when it changes",
  code: "import { motion, AnimatePresence } from \"motion/react\"\n\ntype Props = { home: number; away: number; label: string }\n\nexport default function App({ home, away, label }: Props) {\n  return (\n    <div className=\"flex h-full w-full items-center gap-6 rounded-2xl bg-ink/90 px-8 text-white\">\n      <span className=\"text-2xl font-semibold tracking-widest text-brand\">{label}</span>\n      <AnimatePresence mode=\"popLayout\">\n        <motion.span key={`${home}-${away}`} initial={{ y: 24, opacity: 0 }} animate={{ y: 0, opacity: 1 }} exit={{ y: -24, opacity: 0 }} className=\"text-6xl font-extrabold tabular-nums\">{home} – {away}</motion.span>\n      </AnimatePresence>\n    </div>\n  )\n}\n",
  styles: "@theme { --color-brand: #F43F5E; --color-ink: #0B0F1A; }"
} })
```
A layer shows it, and passes its props — each a literal or bound like any prop (skill binding-live-data):
```jsx
<Video id="bug" source="scoreBug" x={96} y={54} w={560} h={120} props={{ home: state.score.home, away: state.score.away, label: "LIVE" }} />
```

## The code
- `export default function App(props: Props)`, with `Props` declared in the file. It is type-checked (strict) and compiled when set; an error comes back with its line — fix it and set again (`edits: [{ field: "code", old, new }]` for small changes).
- Imports: `react` (hooks) and `motion/react` (motion, AnimatePresence, useAnimate, …). Nothing else.
- The network works as for any web page (public addresses only): fonts, pictures and fetches by URL load. Prefer inline SVG, `data:` URLs and system fonts (`font-sans`, `font-mono`) when they do: nothing to wait for on air. The show's data comes in through props, not fetches.
- Style with Tailwind v4 class names written out in full (`bg-rose-500`; a class built at run time like `bg-${c}-500` is not generated — use `style={{ … }}` for computed values). `styles` holds Tailwind CSS: `@theme` colours and fonts, `@keyframes`, plain rules.
- The page is exactly the layer's box (w×h, CSS px): fill it with `h-full w-full`. The background is transparent — paint only what should show.
- New props re-render the page (its state stays). Animate a change with a motion `key` (above) or `animate={{ … }}`.
- One page per source: two layers showing it share the first layer's props. Different props → another source with the same code.
- A page that throws shows the error in red: check the picture.

## Check
get the frame `as: 'image'`: the page is drawn in its layer, with the layer's current props.

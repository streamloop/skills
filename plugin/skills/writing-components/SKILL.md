---
name: writing-components
description: "Writes reusable components for a Streamloop scene through the Streamloop MCP — a lower third, a score bug, a card — as TSX with a props schema, so its frames place them like elements. Use when someone asks for a component in their Streamloop scene, or the same design repeats across its frames. Not for React components outside Streamloop."
---
# Writing components

> With the Streamloop MCP: every `scene_*` tool also takes `scene`, the scene's id (`scn_…`) — a call written `scene_get('frame/intro', { as: 'image' })` is `scene_get { scene, path: "frame/intro", as: "image" }`. Changing or removing something that exists also takes its `revision` (a layer: its own or its frame's; `"*"` overwrites on purpose), or the answer is REVISION_REQUIRED. Every set changes the draft only: viewers see it after `publish_scene { scene, draftRevision }` (the `draftRevision` every answer carries). Starting one from nothing: skill building-scenes.

```
scene_set('component/NameTag', { value: {
  description: "The guest's name and title on a dark card",
  props: { name: { type: "string", default: "Guest", description: "Big line" }, title: { type: "string", default: "Title", description: "Small line" } },
  tsx: "export default function NameTag(p: Props) {\n  return <Stack direction=\"column\" gap={4} padding={[16, 24]} style={{ background: \"#111827\", borderRadius: 16 }}>\n    <Text value={p.name} prop=\"name\" style={{ fontSize: 44, fontWeight: 800, color: \"#FFFFFF\" }} />\n    <Text value={p.title} prop=\"title\" style={{ fontSize: 22, color: \"#9CA3AF\" }} />\n  </Stack>\n}"
} })
```
- `props`: name → JSON Schema with a `default` and a `description`; colours `{ type: "string", format: "color" }`.
- `tsx`: no imports; `export default function Name(p: Props)`; `p` has every prop, defaults filled in. It returns elements written as in a frame, without ids (skill composing-frames). `prop="name"` on the element that shows a prop lets the operator edit it by clicking it.
- Hooks: `useData("id")` (a table: status, rows), `useSource("id")` (a media source: status, track), `useState()` (show state), `useTicker("8s")` (whole intervals since the frame went on air — rotate with `useTicker("8s") % items.length`), `useBox()` (its own size), `useTokens()`.
- Code shape: `const`s and one `return <…/>`. Expressions: `?:`, `&&`, arithmetic, template strings, `.map/.find/.filter` on arrays. No `if`/`for`/`switch`. Calls: the hooks, and the same functions a frame's bindings use — `String`, `upper`, `lower`, `truncate`, `number(v, "$0,0.00")`, `percent`, `date`, `time`, `fallback`, `slice(list, 1, 4)` — never `Math.*` or methods like `toFixed` (`number(v, "0")` rounds). `useData()` rows are untyped; prefer typed props bound by the frame.
- Styles are the elements' (scene_get('element')): a box takes `background`, `borderColor`, `borderWidth`, `borderRadius`, `boxShadow`; text takes `fontFamily`, `fontWeight` (300–900), `fontSize`, `fontStyle`, `color`, `textAlign`, `lineHeight`, `letterSpacing`. Nothing else from CSS: no `margin` in `style` (inside a Stack a child takes `margin`, `grow`, `alignSelf`, `minW`/`maxW` as props, beside the Stack's `gap`/`padding`), no `objectFit` (Image has `fit`). Stack `justify`: `start` · `center` · `end` · `stretch` · `between` · `around` · `evenly`; `align`: `start` · `center` · `end` · `stretch` · `baseline`.
- Pure: the same props and show give the same output. No network, timers or randomness.
- A frame places it: `<NameTag id="tag" x={96} y={860} name={controls.guestName} title={controls.guestTitle} />`.

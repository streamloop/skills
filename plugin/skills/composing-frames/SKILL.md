---
name: composing-frames
description: "Lays out and restyles the frames of a Streamloop scene through the Streamloop MCP — structure, layout, colour, type and broadcast conventions — from a description or a reference picture, checked with a picture of the result. Use for visual work on a Streamloop frame: moving, resizing or aligning layers, \"make it fit\", \"make it look better\", matching a screenshot, applying the scene's colours. Starting a scene from nothing: building-scenes."
---
# Composing frames

> With the Streamloop MCP: every `scene_*` tool also takes `scene`, the scene's id (`scn_…`) — a call written `scene_get('frame/intro', { as: 'image' })` is `scene_get { scene, path: "frame/intro", as: "image" }`. Changing or removing something that exists also takes its `revision` (a layer: its own or its frame's; `"*"` overwrites on purpose), or the answer is REVISION_REQUIRED. Every set changes the draft only: viewers see it after `publish_scene { scene, draftRevision }` (the `draftRevision` every answer carries). Starting one from nothing: skill building-scenes.

## Workflow
1. New frame: pick an id that isn't taken (scene_get('frame/*') lists them). Existing frame: get it (its code and revision).
2. Decide the structure before any coordinates: which things move together? Rows, columns, cards, badges and anything with text → a Stack (it flows its children). Layers that keep their own x/y but should show, hide or move together → a Group, or a fragment `<>…</>` under one condition. One-off placement on the frame → an element with x/y.
3. Write the code and set it: `edits: [{ old, new }]` for a small change, `value: { name, description, code }` for a new or rewritten frame (send all three on a rewrite: `value` is the whole frame). The stored code is re-indented on every set; `old` matches ignoring whitespace, so edit from the code you last read without re-reading it.
4. Check once: get the frame `as: 'image'` (overlay `ids` outlines the layers). The answer lists `drawn`, `notDrawn` and `issues` (cut-off text, outside title-safe, overlaps, text too small). Fix the issues, then stop.

## The code
`scene_set('frame/guestIntro', { value: { name: "Guest intro", description: "The guest's name over the studio feed", code } })` with `code`:
```jsx
<Frame background={{ type: "video", source: "studioFeed" }}>
  <Stack id="strap" x={96} y={836} direction="column" gap={4} padding={[20, 32]} style={{ background: "#0B0F1AE6", borderRadius: 16 }} motion={{ enter: { kind: "rise", ms: 420 } }}>
    <Text id="guestName" value={controls.guestName} style={{ fontSize: 56, fontWeight: 800, color: "#FFFFFF" }} />
    <Text id="guestTitle" value={controls.guestTitle} style={{ fontSize: 30, color: "#FFFFFFB3" }} />
  </Stack>
</Frame>
```
- `code` is the `<Frame>` element alone: the frame's id is its path. Layer ids are unique in the whole show; keep existing ones. Name layers with `layer={{ name, description }}`.
- Backgrounds: `{ type: "solid", color }` · `{ type: "gradient", from, to, angle }` · `{ type: "image", src }` · `{ type: "video", source }`, with literal values (no bindings or tokens; for a token colour, put a full-frame Rect first).
- Elements: Stack, Group, Rect, Text, Number, Clock, Ticker, Image, Video, Lottie, Aurora, Audio, and the show's components. Their props: scene_get('element/<Type>'); the box props all share: scene_get('element').
- A picture from the web: give its URL as `src`. Setting the frame imports it into the show (the answer names the file, `uploads/…`) and the code names the file from then on — on air the channel renders files, never URLs. A URL the studio can't fetch is refused: use another picture or attach the file. A picture the workspace already has as an upload: its `downloadUrl` (from `get_upload`) is such a URL.
- `style` is not CSS. A box takes `background`, `borderColor`, `borderWidth`, `borderRadius`, `boxShadow`; text adds `fontFamily`, `fontWeight`, `fontSize`, `fontStyle`, `color`, `textAlign`, `lineHeight`, `letterSpacing`. `style` has no margin or per-side borders: inside a Stack a child takes `margin` (px, `[vertical, horizontal]` or `[top, right, bottom, left]`), `grow`, `alignSelf`, `minW`/`maxW` as props next to the Stack's `gap`/`padding`; a rule is a thin Rect.
- Live values (`state.…`, `data.…`, `sources.…`, repeats, conditions): skill binding-live-data.

## Layout
- Design px from the parent's top left. The frame is the show's size (1920×1080 unless the scene says otherwise).
- Inside a Stack, x/y are ignored: the Stack places its children. `justify` works along `direction`, `align` across it. Wrapping existing layers in a Stack re-flows them: to wrap without moving anything, use a Group (children keep their x/y) or a fragment.
- A Stack without w/h hugs its content. Give it a size to align or justify inside a fixed box.
- No right/bottom props: anchor by arithmetic (`x = 1920 - 96 - w`), or use a full-frame Stack (`w={1920} h={1080} padding={[54, 96]}`) with `justify="end"` / `align="end"`.
- Text is one line unless it has `width`; give text that can grow `width` and `maxLines`.
- Card walls: `<Stack layout="grid" columns={3} gap={24} w={1728}>` around a repeat.

## Look
- Paint and type go in `style={{ … }}` with CSS names: background, borderColor, borderWidth, borderRadius, boxShadow; color, fontFamily, fontSize, fontWeight, textAlign, lineHeight, letterSpacing.
- If the show has tokens (scene_get('tokens')), use them: `style={{ background: tokens.colors.accent }}`. Hardcode only one-offs.

## Broadcast conventions (1920×1080)
- Keep text title-safe: 96 px from the left and right edges, 54 px from the top and bottom.
- Lower third: bottom left, y ≈ 800–980, 700–1200 wide; name 48–64 bold, title 28–36.
- Logo, clock, LIVE badge: top corners, inside title-safe.
- Crawl (ticker bar): full width at the bottom, 64–80 tall, its bottom edge at y ≤ 1026: a row Stack with the bar's background, an optional label, then `<Ticker w="fill" …/>` scrolling the items — never a Text that has to fit them all. Speed 80–160 px/s; a separator in the accent colour reads well.
- Text at least 24 px; body 32+; headlines 64+. Text over video sits on a panel or has a shadow.
- Enter motion 300–600 ms: rise or fade for straps, wipe for bars.

## From a reference picture
Scale its size onto the frame (1280×720 → ×1.5), find the structure (bars, cards, rows), then measure each part. Compare with the picture of your frame, overlay `grid`.

More layouts (split screen, card wall, crawl, bug): [examples.md](examples.md).

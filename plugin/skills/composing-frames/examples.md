# Common layouts (1920×1080)

## Split screen: two feeds with name bars
```jsx
<Stack id="split" x={96} y={180} w={1728} h={720} direction="row" gap={24}>
  <Group id="left" w="fill" h={720} clip={true} style={{ borderRadius: 16 }}>
    <Video id="leftFeed" source="studioFeed" w={852} h={720} />
    <Text id="leftName" x={24} y={660} value="Studio" style={{ fontSize: 32, fontWeight: 700, color: "#FFFFFF" }} />
  </Group>
  <Group id="right" w="fill" h={720} clip={true} style={{ borderRadius: 16 }}>
    <Video id="rightFeed" source="guestCam" w={852} h={720} />
    <Text id="rightName" x={24} y={660} value={controls.guestName} style={{ fontSize: 32, fontWeight: 700, color: "#FFFFFF" }} />
  </Group>
</Stack>
```
Two `w="fill"` children share the row equally: (1728 − 24) / 2 = 852 each.

## Card wall from a table
```jsx
<Text id="heading" x={96} y={72} value="Headlines" style={{ fontSize: 64, fontWeight: 800, color: "#FFFFFF" }} />
<Stack id="cards" x={96} y={200} w={1728} layout="grid" columns={3} gap={24}>
  {data.stories.rows.map((row) => (
    <Stack id="card" key={row.id} max={6} h={300} gap={16} padding={32} style={{ background: "#FFFFFF10", borderRadius: 24 }}>
      <Text id="cardTitle" value={row.title} width={496} maxLines={2} style={{ fontSize: 36, fontWeight: 700, color: "#FFFFFF" }} />
      <Text id="cardBody" value={row.summary} width={496} maxLines={4} style={{ fontSize: 24, color: "#FFFFFFA6", lineHeight: 1.35 }} />
    </Stack>
  ))}
</Stack>
```
Column width = (1728 − 2 × 24) / 3 = 560; text width = 560 − 2 × 32 padding = 496.

## Crawl (ticker bar) from a table
```jsx
<Stack id="tickerBar" x={0} y={950} w={1920} h={76} direction="row" align="center" style={{ background: "#0B0F1AF2" }} motion={{ enter: { kind: "wipe", ms: 500 } }}>
  <Stack id="tickerLabel" h={76} justify="center" padding={[0, 28]} style={{ background: "#E63946" }}>
    <Text id="tickerLabelText" value="BREAKING" style={{ fontSize: 30, fontWeight: 800, color: "#FFFFFF", letterSpacing: 4 }} />
  </Stack>
  <Ticker id="crawl" w="fill" h={76} items={data.stories.rows} field="title" speed={120} separatorColor="#E63946" style={{ fontSize: 32, fontWeight: 600, color: "#FFFFFF" }} />
</Stack>
```
The bar ends at 950 + 76 = 1026 (title-safe); the crawl fills the row after the label and scrolls under its edge. `items` takes a table's rows (each shows `field`), a list (`items={["Doors open 7pm", "Free parking"]}`) or one text (`items={controls.breakingNews}`). Rows that change join at the end of the loop. `direction="right"` for right-to-left languages; `separator`, `separatorColor`, `gap` set what sits between items.

## Corner bug: logo and clock, top right
```jsx
<Stack id="bug" x={1584} y={54} w={240} direction="row" justify="end" align="center" gap={16}>
  <Image id="logo" src="logo.png" w={64} h={64} fit="contain" />
  <Clock id="clock" format="HH:mm" style={{ fontSize: 36, fontWeight: 700, color: "#FFFFFF" }} />
</Stack>
```
x = 1920 − 96 (title-safe) − 240 (its width) = 1584.

---
name: calling-the-streamloop-api
description: Writes code and curl commands against Streamloop's HTTP APIs — the REST API (streams, playlists, uploads, destinations, schedules, billing) and the Scenes API (scenes, their code, pictures, publishing) — with API-key or OAuth auth, workspaces, cursor pagination, idempotent retries, revisions and RFC 7807 errors. Use whenever someone wants a script, backend, cron job, CI step, Zapier/n8n flow or app that controls Streamloop without the MCP, e.g. "start my stream from a script", "upload every file in this folder", "update the scoreboard from my server", "publish the scene from CI", or asks how to authenticate or paginate.
---
# Calling Streamloop's HTTP APIs

Two APIs on one host, `https://api.streamloop.app`, with the same conventions. Each publishes its OpenAPI document: read it for exact fields instead of guessing.
- **REST** — `/v1/streams`, `/v1/playlists`, `/v1/uploads`, `/v1/destinations`, `/v1/billing/*`, `/v1/workspaces`, `/v1/me`. Spec: `GET /v1/openapi.json`.
- **Scenes** — `/v1/scenes…`: a scene (`scn_…`) and everything in it, read and written by path. Spec: `GET /v1/scenes/openapi.json`.

## Auth and workspace
- Your own scripts use an **API key** (`sl_…`, created in the dashboard), sent as `X-API-Key: sl_…`. Keep it server-side, read it from an environment variable, and never write it into code or a browser bundle.
- An API key carries no workspace, so every request names one in `X-Workspace-Id: wksp_…`. The one route that needs no header is the bootstrap: `GET /v1/workspaces` answers a plain array of the key's workspaces (id, name, your role); pick one and send its id from then on. Without the header every route that acts in a workspace answers 400 `WORKSPACE_REQUIRED`; the OpenAPI documents and `GET /v1/status` are public and need neither header nor key.
- Apps acting for other users use OAuth 2.1 with PKCE (`https://auth.streamloop.app`) and send `Authorization: Bearer …`. Ask for the narrowest scopes: `streamloop:read`, `:write`, `:destructive`.

## Conventions you can rely on
- **Errors** are `application/problem+json`: `{ type, title, status, detail, code }`, plus what the code needs (`reason`, `errors[]`, `currentRevision`, …). Branch on `code` (`NOT_FOUND`, `CONFLICT`, `STREAM_LIVE`, …), not on `detail`, which is prose for people.
- **Lists** of resources return `{ data, page: { nextCursor } }`. Pass `?limit=` (1–100) and `?cursor=<nextCursor>` until `nextCursor` is null. (`GET /v1/workspaces` is the exception: a plain array, no paging.)
- **Retries:** send `Idempotency-Key` on every POST that does something (create, start, stop, publish). A retry with the same key replays the first answer for 24 h instead of doing the work twice; the same key with another body is refused. One key per logical operation, not per attempt — for a job that may run twice, derive it from the job (`start-stream_01ABC-2026-10-08`), not from a fresh uuid. A 4xx is replayed too: after fixing what a 409 or 412 refused, send the corrected request under a new key (a merge patch retried after a 412 needs none; it is idempotent by itself). A 5xx is not remembered.
- **Rate limits:** 600 requests a minute per key. Read `RateLimit-Remaining` and `RateLimit-Reset`; a 429 (`RATE_LIMITED`) carries `Retry-After` in seconds: wait that long, then retry the same request with the same `Idempotency-Key`.
- **Stream state** (`GET /v1/streams/{id}` → `state`): `draft`, `invalid`, `scheduled`, `preparing`, `activating`, `active`, `failing`, `failed`, `stopping`, `stopped`. "Live" is `preparing`, `activating` or `active`. A start is allowed from `draft`, `scheduled` (it starts now instead of at its window), `stopped`, `failed` and `invalid`; from any other state it is 409 `STREAM_ILLEGAL_TRANSITION`. A job that must not double-start reads the state first: live means done; `stopping` or `failing` means wait and read again (a start then is the same 409, and says nothing about whether it will run). A 200 can still answer `state: invalid` (refused, nothing started): `GET /v1/streams/{id}/readiness` lists what is wrong.
- **Concurrent edits** (Scenes API): a GET answers an `ETag` (strong; a weak `W/` tag never matches). Send it back as `If-Match` on PUT, PATCH and DELETE — for a layer, its own or its frame's revision. If someone changed the resource since, the answer is 412 `PRECONDITION_FAILED` with `currentRevision`, and nothing is overwritten: re-read, re-apply, retry. `If-None-Match: *` on PUT creates only if absent.

## Scenes API in practice
```bash
H=(-H "X-API-Key: $STREAMLOOP_API_KEY" -H "X-Workspace-Id: $WORKSPACE")
SCENE=$(curl -s "${H[@]}" -X POST https://api.streamloop.app/v1/scenes -H 'content-type: application/json' -H "Idempotency-Key: $(uuidgen)" -d '{"name":"Matchday"}' | jq -r .id)
# a frame in it, written as its code (the JSX dialect: GET …/resources/element/* lists the elements)
curl -s "${H[@]}" -X PUT "https://api.streamloop.app/v1/scenes/$SCENE/resources/frame/main" -H 'content-type: text/jsx' --data-binary @main.jsx
# a merge patch on one layer — e.g. a score from your own server (a Text's or Number's text is props.value); If-Match is the ETag a GET of that layer answered
ETAG=$(curl -s -D - -o /dev/null "${H[@]}" "https://api.streamloop.app/v1/scenes/$SCENE/resources/frame/main/score" | tr -d '\r' | awk -F': ' 'tolower($1)=="etag"{print $2}')
curl -s "${H[@]}" -X PATCH "https://api.streamloop.app/v1/scenes/$SCENE/resources/frame/main/score" -H "If-Match: $ETAG" -H 'content-type: application/merge-patch+json' -d '{"props":{"value":"2 – 1"}}'
curl -s "${H[@]}" "https://api.streamloop.app/v1/scenes/$SCENE/resources/frame/main?as=image" -o main.png   # look before publishing
# publish exactly the draft you checked: its revision is the Draft-Revision header (and draftRevision in every draft answer)
REV=$(curl -s -D - -o /dev/null "${H[@]}" "https://api.streamloop.app/v1/scenes/$SCENE" | tr -d '\r' | awk -F': ' 'tolower($1)=="draft-revision"{print $2}')
curl -s "${H[@]}" -X POST "https://api.streamloop.app/v1/scenes/$SCENE/publish" -H 'content-type: application/json' -d "{\"draftRevision\":\"$REV\",\"dryRun\":true}"   # lists what would change, publishes nothing
curl -s "${H[@]}" -X POST "https://api.streamloop.app/v1/scenes/$SCENE/publish" -H 'content-type: application/json' -H "Idempotency-Key: publish-$SCENE-$REV" -d "{\"draftRevision\":\"$REV\"}"   # the publish itself
```
Every write is checked the way the studio checks it. A refused write changes nothing, and its problem lists `errors[]` with lines or layer ids. Edits change the **draft**: viewers see nothing until `POST …/publish`, which requires the `draftRevision` you checked, so a studio user's half-done edit is never published by your job. A `409 STALE_PUBLISH` means someone published since that draft was based: publishing it would take their version off air. Re-read and rebuild; send `overwritePublished` only when a person decided to. If a stream has the scene on air, publish needs `"confirm": true`, because viewers see the change at once. To put a scene on a stream: `PUT /v1/streams/{id}/scene { "sceneId": "scn_…" }`.

Data that changes often (scores, prices, headlines) belongs in a control or a table, not in a republished layout. While a stream plays the scene, `POST /v1/scenes/{id}/live` sets a control (`setControl`), replaces a table's rows (`setData`) or takes a frame to air (`goFrame`), and viewers see it at once with nothing to publish. It answers 409 `SCENE_OFFLINE` when nothing plays the scene. A table can also read a URL your server serves. Republishing a whole scene for each new value is slow and noisy.

## REST in practice
A 24/7 loop is: create the stream → upload media (`POST /v1/uploads` → `POST /v1/uploads/{id}/upload-url` answers a signed URL → PUT the bytes there with the upload's `mimeType` as Content-Type → `POST /v1/uploads/{id}/complete`) → `POST /v1/playlists/{id}/operations` → add a destination → `GET /v1/streams/{id}/readiness` → `POST /v1/streams/{id}/start`. Starting, stopping and deleting affect what the public sees. In code meant for production, make those explicit calls the user chose, not side effects of a sync job.

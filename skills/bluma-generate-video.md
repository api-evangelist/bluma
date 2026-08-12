---
name: bluma-generate-video
description: >-
  Generate a short-form video with the Bluma API — pick a template, submit a prompt,
  track the asynchronous job to completion, and fetch a signed download URL before it
  expires. Use when an agent needs to produce a TikTok/Reels/Shorts-style video
  programmatically.
api: Bluma API v1
base_url: https://api.getbluma.com/api/v1
operations:
  - GET /v1/templates
  - POST /v1/videos
  - GET /v1/videos/{id}
  - GET /v1/videos/{id}/download
  - GET /v1/credits/balance
scopes:
  - templates:list
  - videos:create
  - videos:read
  - videos:download
  - credits:read
generated: '2026-08-12'
method: generated
source: >-
  https://docs.getbluma.com/quickstart , https://docs.getbluma.com/concepts/templates ,
  https://docs.getbluma.com/concepts/credits , https://docs.getbluma.com/errors
grounding: >-
  Bluma publishes no readable OpenAPI (its advertised spec at
  https://api.getbluma.com/api/v1/openapi.json returns 401), so operations are identified
  by HTTP method and path rather than by operationId. Every path, field, status code and
  id prefix used here is copied verbatim from Bluma's own documentation; none is inferred.
---

# Generate a video with Bluma

Bluma renders short-form video asynchronously. You submit a job, it returns immediately
with an id, and the finished asset arrives minutes later. Never treat `POST /v1/videos`
as if it returns a video.

## Before you start

Authenticate every request with a bearer API key:

```
Authorization: Bearer bluma_test_...   # test: free, watermarked, 720p max
Authorization: Bearer bluma_live_...   # production: charged, full quality
```

Develop against a `bluma_test_` key. Test mode is free, unlimited, and exercises an
identical API surface — the only differences are a "TEST MODE" watermark, a 720p cap,
24fps, and low-priority queueing.

## Steps

### 1. Check the credit balance first

```
GET /v1/credits/balance
```

Returns `credits`, `tier`, `monthly_allowance`, `overage_used` and `reset_date`. A
production render costs **5 credits per video** for every template and every user
variant. If the balance is short, the create call fails with `402 insufficient_credits`
**after** you have already spent a round trip — check first.

Free-tier accounts have no overage: a 402 there is terminal until the plan is upgraded.

### 2. List the templates

```
GET /v1/templates
```

Each entry carries `id`, `name`, `description`, `category`, `credits_per_video` and
`duration_range`. Template ids are human-readable slugs, not opaque ids —
`consumerclub-discord-zoomed`, `cat-explainer-v2`, `rick-morty-explainer`,
`steel-griffin-explainer`.

Cache this list. It changes rarely, and Bluma's own rate-limit guidance names it as the
call worth caching.

Use `GET /v1/templates/{id}` when you need `ai_features` (whether the template does
script, voice and/or image generation).

### 3. Submit the job

```
POST /v1/videos
Content-Type: application/json

{
  "template_id": "consumerclub-discord-zoomed",
  "context": {
    "prompt": "Create a Discord conversation about discovering a great product"
  },
  "webhook_url": "https://yourapp.com/webhooks/video-complete"
}
```

- `context.prompt` must be at least 10 characters — shorter fails with
  `400 invalid_request` and a `validation_errors[]` array naming `context.prompt`.
- `context.brand_id` optionally applies a saved brand identity (colors, logo, fonts).
- `context.custom_data` carries template-specific overrides on templates that accept them.
- `webhook_url` is optional but strongly preferred over polling.

The response is the accepted job, not the video:

```json
{
  "id": "batch_abc123xyz",
  "status": "queued",
  "template_id": "consumerclub-discord-zoomed",
  "created_at": "2025-11-03T10:30:00Z",
  "estimated_completion": "2025-11-03T10:32:00Z",
  "status_url": "/v1/videos/batch_abc123xyz",
  "credits_charged": 2
}
```

**Critical: this call is not idempotent.** Bluma documents no `Idempotency-Key` header
and no request-deduplication behavior. A retry after a network timeout can start a second
render and charge credits twice. If a create call times out, do **not** blindly retry —
treat the outcome as unknown and reconcile before resubmitting.

### 4. Wait for completion

Prefer webhooks. Subscribe to `video.completed` and `video.failed` and let Bluma call you
— see the `bluma-handle-webhooks` skill. Webhook deliveries are exempt from rate limits.

If you must poll:

```
GET /v1/videos/batch_abc123xyz
```

Poll no faster than **every 5 seconds** (the interval in Bluma's own samples). Typical
generation takes **2-5 minutes**. `status` moves through `queued` → `processing` →
`completed` | `failed`, with a `progress` integer from 0 to 100.

Every poll counts against the hourly rate limit. Watch `X-RateLimit-Remaining` and stop
polling when it drops near zero.

### 5. Download before the URL expires

```
GET /v1/videos/batch_abc123xyz/download
```

```json
{
  "download_url": "https://cdn.getbluma.com/videos/batch_abc123xyz.mp4?signed=...",
  "expires_at": "2025-11-03T11:31:45Z"
}
```

The signed URL lasts **one hour**. Fetch the bytes immediately and store them yourself;
do not persist the signed URL as if it were permanent. If it has expired, call the
download endpoint again for a fresh one.

The completed job body also carries a `url`, `thumbnail_url`, `duration` and
`size_bytes`.

## Error handling

| Status | type | What to do |
|---|---|---|
| 400 | `invalid_request` | Read `validation_errors[]` and fix each named field. Do not retry unchanged. |
| 401 | `authentication_error` | Key missing, invalid or revoked. Do not retry. |
| 402 | `insufficient_credits` | `metadata` gives `credits_required` / `credits_available`. Top up or downgrade the render. Do not retry. |
| 403 | `permission_denied` | `metadata.required_scope` names what the key lacks. Issue a key with that scope. |
| 404 | `not_found` | Wrong id, or the job belongs to another account. |
| 429 | `rate_limit_exceeded` | Honor `Retry-After` (seconds), then exponential backoff. |
| 5xx | `internal_error` / `service_unavailable` | Retry with exponential backoff. Log `request_id` and quote it to support. |

The documented envelope nests everything under `error`:
`error.type`, `error.title`, `error.status`, `error.detail`, sometimes `error.metadata`
and `error.request_id`. Bluma's docs call this RFC 7807; it is not — it is not
`application/problem+json` and the fields are not at the top level, so generic problem-
details parsers will fail. Worse, the unauthenticated edge returns a **bare string**:
`{"error":"No authorization token provided"}`. Guard against `error` being a string
before reading `error.type`.

A render that fails *after* acceptance never produces an HTTP error at all — it arrives
as a `video.failed` webhook with `data.error.type` and `data.error.detail`. Credits are
refunded automatically on server-side render failure.

## Cost model

Every template and every user-created variant costs **5 credits per video**, covering
script generation, voice, images, and rendering. Bluma's credits page also publishes an
older multiplier formula (base × duration × resolution × AI multiplier, with 720p/1080p/4K
at 1.0×/1.5×/3.0×); the two models are not reconciled in Bluma's own documentation. Read
`credits_charged` on the create response as the authoritative per-job figure.

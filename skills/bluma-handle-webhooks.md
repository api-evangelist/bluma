---
name: bluma-handle-webhooks
description: >-
  Register a Bluma webhook, verify its HMAC signature, deduplicate deliveries, and react
  to the seven published video and credit events. Use instead of polling whenever an
  agent needs to know when a Bluma render finishes or fails.
api: Bluma API v1
base_url: https://api.getbluma.com/api/v1
operations:
  - POST /v1/webhooks
  - GET /v1/webhooks
  - DELETE /v1/webhooks/{id}
  - GET /v1/webhooks/{id}/deliveries
scopes:
  - webhooks:manage
generated: '2026-08-12'
method: generated
source: >-
  https://docs.getbluma.com/concepts/webhooks , https://docs.getbluma.com/guides/webhooks-setup ,
  https://docs.getbluma.com/errors
grounding: >-
  Bluma publishes no AsyncAPI and no readable OpenAPI, so operations are identified by
  HTTP method and path. Every event name, header, retry interval and payload field below
  is copied verbatim from Bluma's webhooks documentation.
---

# Handle Bluma webhooks

Video generation takes 2-5 minutes. Polling burns rate limit; webhooks do not — Bluma
exempts webhook deliveries from rate limits entirely.

## 1. Register an endpoint

```
POST /v1/webhooks
Content-Type: application/json

{
  "url": "https://yourapp.com/webhooks/bluma",
  "events": ["video.completed", "video.failed"]
}
```

```json
{
  "id": "webhook_xyz789",
  "url": "https://yourapp.com/webhooks/bluma",
  "events": ["video.completed", "video.failed"],
  "secret": "whsec_...",
  "is_active": true,
  "created_at": "2025-11-03T10:30:00Z"
}
```

**The `whsec_` secret is shown exactly once.** Persist it to your secret store in the
same call that creates the webhook. There is no documented way to retrieve it later.

Manage with `GET /v1/webhooks` and `DELETE /v1/webhooks/{id}`.

## 2. Events

| Event | Fires when |
|---|---|
| `video.queued` | Immediately after creation |
| `video.processing` | When rendering begins |
| `video.completed` | Generation succeeded |
| `video.failed` | An error occurred during rendering |
| `video.deleted` | A user deleted the video |
| `credits.low` | Balance drops below 10 |
| `credits.exhausted` | Balance reaches 0 |

Subscribe to `credits.low` and `credits.exhausted` on any automated pipeline. They are
the only warning before renders start failing with `402`.

## 3. Envelope

```json
{
  "id": "evt_abc123xyz",
  "type": "video.completed",
  "created_at": "2025-11-03T10:31:45Z",
  "data": { }
}
```

`video.completed` `data`: `id`, `status`, `template_id`, `url`, `thumbnail_url`,
`duration`, `size_bytes`, `credits_consumed`.

`video.failed` `data`: `id`, `status`, `template_id`, `error.type`, `error.detail`.

Headers on every delivery:

```
X-Bluma-Signature: sha256=<hex>
X-Bluma-Event-Id: evt_abc123xyz
X-Bluma-Event-Type: video.completed
User-Agent: Bluma-Webhooks/1.0
Content-Type: application/json
```

## 4. Verify the signature — always

HMAC-SHA256 over the **raw request body**, keyed with the `whsec_` secret, compared
against `X-Bluma-Signature` in the form `sha256=<hex>`.

```js
const expected = crypto
  .createHmac('sha256', process.env.BLUMA_WEBHOOK_SECRET)
  .update(rawBody)          // raw bytes — never a re-serialized JSON object
  .digest('hex');

if (signature !== `sha256=${expected}`) return res.status(401).send('Invalid signature');
```

Both SDKs ship a helper: `Bluma.webhooks.verify(payload, signature, secret)`.

Two rules that cause almost every verification failure:

1. Compute the HMAC on the **raw body**, before any JSON parsing. Mount the route with a
   raw body parser (`express.raw({ type: 'application/json' })`).
2. Use the secret from the webhook that received this delivery, not a different one.

The signature covers the body only — there is **no timestamp and no replay window**. A
captured delivery stays valid forever, so deduplication is your only replay defense.

## 5. Deduplicate on `event_id`

Deliveries are at-least-once. Bluma retries, and a retry after your handler already
succeeded (but the response was lost) will arrive again with the same `id`.

Record `event.id` / `X-Bluma-Event-Id` in a durable set before doing any side effect, and
drop anything already present. This is the *only* idempotency in the Bluma integration —
the REST API itself has no `Idempotency-Key` support.

## 6. Respond fast, retry ladder

Return `2xx` promptly and do the work asynchronously. Failure to return 2xx triggers:

| Attempt | Delay |
|---|---|
| 1st retry | 3 seconds |
| 2nd retry | 30 seconds |
| 3rd retry | 5 minutes |
| 4th retry | 1 hour |

After 4 failed attempts the delivery is marked failed. After **10 consecutive failures**
the webhook is **automatically disabled** and must be re-enabled once your endpoint is
fixed — monitor for this, or you will silently stop receiving events.

## 7. Audit deliveries

```
GET /v1/webhooks/webhook_xyz789/deliveries
```

Returns `id`, `event_id`, `event_type`, `attempt_number`, `status_code`, `duration_ms`,
`error_message` and `created_at` per attempt. Use it to confirm whether a missing event
was never sent or never accepted.

## 8. Testing

Webhooks are fully functional with `bluma_test_` keys. Bluma's guide suggests
webhook.site for a throwaway endpoint and ngrok for tunneling to a local handler.

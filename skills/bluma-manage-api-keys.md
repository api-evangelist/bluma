---
name: bluma-manage-api-keys
description: >-
  Create, list, rotate and revoke Bluma API keys, and move an integration from test to
  production safely. Use when provisioning credentials, doing zero-downtime key rotation,
  or responding to a leaked key.
api: Bluma API v1
base_url: https://api.getbluma.com/api/v1
operations:
  - POST /v1/api-keys
  - GET /v1/api-keys
  - POST /v1/api-keys/{id}/rotate
  - DELETE /v1/api-keys/{id}
generated: '2026-08-12'
method: generated
source: >-
  https://docs.getbluma.com/authentication , https://docs.getbluma.com/guides/test-vs-production ,
  https://docs.getbluma.com/concepts/rate-limits
grounding: >-
  Bluma publishes no readable OpenAPI, so operations are identified by HTTP method and
  path. Every path, field, scope name and prefix below is copied verbatim from Bluma's
  authentication and test-vs-production documentation.
---

# Manage Bluma API keys

## Key shapes

| Prefix | Environment | Behavior |
|---|---|---|
| `bluma_test_` | test | Free and unlimited, no credits charged, 720p max, "TEST MODE" watermark, 24fps, low-priority queue |
| `bluma_live_` | production | Credits charged, up to 4K, no watermark, priority processing. Requires a paid plan. |

Both are presented the same way: `Authorization: Bearer <key>`. The API surface is
identical in both modes — code that works in test works in production.

## A caveat before you automate

The key-management operations are documented as authenticated with a **session token**
(`Authorization: Bearer YOUR_SESSION_TOKEN`), not with an API key. Bluma never documents
how to obtain a session token outside the dashboard, so fully headless key provisioning
is not described by the published material. Plan on creating the first key by hand at
`https://app.getbluma.com/settings?tab=api`.

## Create

```
POST /v1/api-keys
Content-Type: application/json

{
  "name": "Production Server Key",
  "environment": "production",
  "rate_limit_per_hour": 1000
}
```

```json
{
  "api_key": "bluma_live_...",
  "id": "key_abc123",
  "name": "Production Server Key",
  "environment": "production",
  "prefix": "bluma_live_...",
  "scopes": ["videos:create", "videos:read", "templates:list"],
  "rate_limit_per_hour": 1000,
  "created_at": "2025-11-03T10:30:00Z"
}
```

**The secret is returned exactly once.** Write it to your secret store in the same
operation. `GET /v1/api-keys` afterwards returns only the truncated `prefix`.

`rate_limit_per_hour` is optional and defaults to the plan tier. Setting it *lower* is
the documented way to exercise 429 handling — create a test key with
`rate_limit_per_hour: 10` and exceed it.

## Scopes

New keys are issued with a default scope set. The eight published scopes are:

`videos:create`, `videos:read`, `videos:download`, `templates:list`, `templates:read`,
`credits:read`, `webhooks:manage`, `usage:read`.

A call outside the key's scopes returns `403 permission_denied` with
`metadata.required_scope` and `metadata.available_scopes` naming exactly what is missing.

Custom (reduced or extended) scope sets are Enterprise-only — contact
`support@getbluma.com`. On Free, Starter and Pro there is no documented way to issue a
least-privilege key, so treat every key as carrying the full default authority.

## Rotate — the zero-downtime path

```
POST /v1/api-keys/key_abc123/rotate
```

Issues a new key and schedules the old one to expire in **30 days**. Sequence:

1. Rotate; capture the new secret.
2. Deploy the new secret everywhere.
3. Confirm traffic has moved (watch for 401s on the old prefix).
4. Let the old key lapse, or `DELETE` it early once you are certain.

Bluma recommends rotating every **90 days**.

## Revoke — the incident path

```
DELETE /v1/api-keys/key_abc123
```

Immediate and total: every in-flight and future request on that key fails. Have the
replacement key deployed *before* you revoke, unless you are containing a live leak, in
which case revoke first and restore service after.

## Isolate by key, not by account

Rate limits are enforced **per key**, not per account. Bluma presents this as a scaling
lever: issue separate keys for production traffic, batch/background jobs, staging, and
CI, and each gets its own independent hourly budget. It also contains the blast radius of
a leak to one workload.

## Test → production checklist

Bluma's published promotion checklist:

1. Generate with every template you plan to use, on a test key.
2. Exercise error handling — invalid input, 402, 429.
3. Verify webhook signature verification and event handling.
4. Load test to confirm you sit inside the tier's hourly limit.
5. Review output quality, ignoring the watermark.
6. Upgrade the plan at `getbluma.com/billing`.
7. Create the production key and move it into environment variables
   (`BLUMA_API_KEY`, or split `BLUMA_TEST_KEY` / `BLUMA_LIVE_KEY`).
8. Deploy, generate one video to confirm, and subscribe to `credits.low` /
   `credits.exhausted`.

Never hardcode a key, never commit one, and never use a `bluma_live_` key in a test
environment — test renders are free, production renders are not.

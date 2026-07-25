---
name: Subscribe to CompanyCam webhooks and verify deliveries
description: Register a webhook, receive events, and verify the HMAC-SHA1 signature on each delivery.
api: openapi/companycam-openapi-original.yml
operations: [createWebhook, listWebhooks, getWebhook, updateWebhook, deleteWebhook]
---

# Subscribe to CompanyCam webhooks

Use this to receive real-time events (project/photo/comment/document/video/todo_list/task).

## Auth
`Authorization: Bearer <API_TOKEN>`. Base URL `https://api.companycam.com/v2`.

## Steps
1. `createWebhook` with your HTTPS endpoint URL and the event scopes you want (e.g. `photo.created`, `project.*`, or `*` for all). Store the returned webhook `token` — you need it to verify signatures.
2. `listWebhooks` / `getWebhook` to confirm registration and inspect error counts.
3. Receive POSTs at your endpoint. Each body is `{ event_type, created_at, payload, webhook_id }`.
4. Verify every delivery: compute `base64(HMAC-SHA1(raw_request_body, webhook_token))` and timing-safe compare it to the `X-CompanyCam-Signature` header. Reject on mismatch.
5. Return `200 OK` quickly. Non-200 triggers exponential-backoff retries (max 10); a webhook is auto-disabled after 25 accumulated errors.
6. `updateWebhook` to change scopes/URL; `deleteWebhook` to remove it.

## Rules
- Delivery is at-least-once — dedupe on `webhook_id` + event.
- Videos: on `video.created`/`video.updated`, poll `getVideo` until `status: processed` before using `playback_url`/`format` (before then it is the raw upload URL, not the HLS `.m3u8`).
- Full event catalog: asyncapi/companycam-webhooks.yml.

---
name: formbase-requests
description: Ask a customer for information through formbase and read the verified answers back. Use when you need a confirmation, a document, a signature or missing details from one named person.
---

# formbase requests

A request is one form assigned to one recipient: its own link, its own prefilled values, its own expiry, its own outcome. Use `formShareLink_create` instead when anyone may answer.

1. Find the form with `form_list`. Ask the user which one if several match.
2. Call `fields_list` with the form id. It returns every field key with its value type, option keys and a `usage` line saying whether it goes in `prefill` (visible questions) or `context` (hidden fields). Prefill takes option keys, never labels.
3. Call `request_create` with `recipient` (email, name), `prefill` for values you know, `readonly` for prefilled values the recipient must not change, `context` for hidden fields, and `delivery` (`"email"` lets formbase send the invitation, which needs Pro and `recipient.email`; `"none"`, the default, returns the link for you to send). Pass `reminders` (offsets like `["2d","5d"]`, Pro, needs `recipient.email`) when the user asks for them, or `[]` for none; omitting it keeps the form's own schedule. Pass `expiresAt` (epoch ms, default 30 days) when the user gives a deadline, and `callbackUrl` if the user has a webhook to receive the answers.
4. Give the user the request id and link. Poll `request_get` when asked, or let the callback tell you. Status moves from `pending` to `completed`, `expired` or `canceled`; a completed request carries `answers` keyed by field key.
5. `request_remind` sends one reminder now (Pro, at most 8 reminders per request, one manual send per 10 minutes). `request_cancel` closes the link and cannot be undone; it is marked destructive, so confirm with the user first. `request_replayCallback` re-sends a callback that never landed.

Do not invent field keys. Re-read `fields_list` after any `form_publish`. Call `load_skill` with `requests` for the full reference.

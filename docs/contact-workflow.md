# Contact Workflow: Node-by-Node

## Workflow Order

```text
Contact Form Webhook
→ Rate Limit
→ Rate Limit Exceeded?
   ├── true  → Respond 429 Too Many Requests
   └── false → Validate and Sanitize
               → Honeypot Spam Detected?
                  ├── true  → Respond 200 Success
                  └── false → Contact Data Valid?
                              ├── false → Respond 422 Validation Error
                              └── true  → Verify Cloudflare Turnstile
                                         → Turnstile Verification Passed?
                                            ├── false → Respond 403 Verification Failed
                                            └── true  → Restore Sanitized Contact Data
                                                       ├── Send Telegram Lead Notification
                                                       └── Store Lead in Google Sheets
                                                            ↓
                                                       Wait for Lead Actions
                                                            ↓
                                                       Respond 200 Success
```

## 1. `Contact Form Webhook`

Receives production and test `POST` requests.

```text
Development: http://localhost:5678/webhook-test/contact
Production:  https://n8n-service-71j0.onrender.com/webhook/contact
```

Allowed browser origin:

```text
https://saurabhzaiswal.vercel.app
```

## 2. `Rate Limit`

Applies a lightweight per-IP limit before validation and external API calls.

```text
2 requests per IP
60-second window
```

It reads the visitor IP from `cf-connecting-ip`, `x-forwarded-for` or `x-real-ip` and stores counters using:

```js
const store = $getWorkflowStaticData("global");
```

Outputs include:

```text
clientIp
rateLimited
requestCount
retryAfterSeconds
```

## 3. `Rate Limit Exceeded?`

Checks:

```js
{{ $json.rateLimited }}
```

```text
true  → Respond 429 Too Many Requests
false → Validate and Sanitize
```

## 4. `Respond 429 Too Many Requests`

Returns HTTP `429` with a retry message and optional `Retry-After` value.

## 5. `Validate and Sanitize`

Normalizes and validates:

```text
name
email
subject
message
website
turnstileToken
timestamp
IP address
user agent
```

It trims values, applies length limits, validates email, sanitizes unsafe content, detects the honeypot and collects errors.

## 6. `Honeypot Spam Detected?`

Checks `isSpam`.

```text
true  → Respond 200 Success
false → Contact Data Valid?
```

Detected bots receive a generic success response but are not stored or notified.

## 7. `Contact Data Valid?`

Checks `isValid`.

```text
true  → Verify Cloudflare Turnstile
false → Respond 422 Validation Error
```

## 8. `Respond 422 Validation Error`

Returns HTTP `422` with a generic validation message and validation errors.

## 9. `Verify Cloudflare Turnstile`

Calls:

```text
POST https://challenges.cloudflare.com/turnstile/v0/siteverify
```

Uses the private Turnstile secret, browser token and visitor IP.

## 10. `Turnstile Verification Passed?`

Checks Cloudflare's `success` value.

```text
true  → Restore Sanitized Contact Data
false → Respond 403 Verification Failed
```

## 11. `Respond 403 Verification Failed`

Returns HTTP `403` when Turnstile validation fails.

## 12. `Restore Sanitized Contact Data`

Restores fields from `Validate and Sanitize` after the Turnstile HTTP node replaces `$json`.

Example:

```js
{{ $('Validate and Sanitize').item.json.name }}
```

Verify all expressions after renaming nodes.

## 13. `Send Telegram Lead Notification`

Sends a private immediate lead alert. Telegram credentials and chat IDs must remain private.

## 14. `Store Lead in Google Sheets`

Appends the verified lead to the configured spreadsheet.

Recommended columns:

```text
Timestamp
Name
Email
Subject
Message
IP
User Agent
Status
Source
```

## 15. `Wait for Lead Actions`

Merge node that waits for Telegram and Google Sheets outputs before the final success response.

## 16. `Respond 200 Success`

Returns the final success JSON. The same generic success response can be used for honeypot spam without storing the submission.

## Security Checklist

- [x] Restricted production CORS origin
- [x] Per-IP request limit
- [x] Server-side validation
- [x] Input sanitization
- [x] Honeypot
- [x] Turnstile verification
- [x] Google Sheets OAuth
- [x] Private Telegram credentials
- [x] `403`, `422` and `429` responses
- [ ] Atomic Redis/PostgreSQL or Cloudflare edge rate limiting
- [ ] Failure-tolerant integration branches
- [ ] Separate error workflow

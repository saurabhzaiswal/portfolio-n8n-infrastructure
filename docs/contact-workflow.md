# Contact Workflow: Node-by-Node

1. **Webhook** — receives production and test requests.
2. **Code in JavaScript** — validates, sanitizes and creates `isSpam` and `isValid`.
3. **Is Spam?** — checks the hidden honeypot field.
4. **Is Valid?** — returns `422` for invalid submissions.
5. **Verify Turnstile** — calls Cloudflare Siteverify.
6. **Is Turnstile Valid?** — returns `403` on failure.
7. **Restore Contact Data** — restores the sanitized form values after the HTTP node replaces `$json`.
8. **Google Sheets** — appends the verified lead.
9. **Telegram** — sends an immediate notification.
10. **Merge** — waits for configured branch inputs.
11. **Respond to Webhook** — returns the final JSON response.

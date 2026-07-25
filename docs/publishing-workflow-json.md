# Publishing an n8n Workflow JSON Safely

Publish only:

```text
workflows/portfolio-contact-form.sanitized.json
```

Keep raw exports outside Git or under `workflows/raw/`.

## Required Placeholders

```text
YOUR_TELEGRAM_CHAT_ID
YOUR_GOOGLE_SHEET_ID
YOUR_TURNSTILE_SECRET_KEY
https://YOUR_PORTFOLIO_DOMAIN.com
```

## Search Before Publishing

```text
secret
token
password
apiKey
authorization
chatId
spreadsheetId
clientSecret
credential
webhookId
email
instanceId
```

Review or remove Telegram IDs, Google Sheet IDs, credential IDs, Turnstile secrets, private emails, test submissions and unnecessary instance metadata.

The rate-limit Code node is safe to publish when it contains no private values, but document that workflow static data is not atomic or distributed.

If a real secret was ever committed, rotate it. Deleting it from the latest commit does not remove it from Git history.

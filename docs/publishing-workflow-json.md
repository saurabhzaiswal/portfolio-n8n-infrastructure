# Publishing an n8n Workflow JSON Safely

Store only the sanitized export in:

```text
workflows/portfolio-contact-form.sanitized.json
```

Keep raw exports outside Git or under `workflows/raw/`.

Before publishing, search for:

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
```

Replace private values with placeholders such as:

```json
{
  "secret": "YOUR_TURNSTILE_SECRET",
  "chatId": "YOUR_TELEGRAM_CHAT_ID",
  "spreadsheetId": "YOUR_GOOGLE_SHEET_ID"
}
```

If a real secret was ever committed, rotate it. Deleting it from the latest commit does not remove it from Git history.

# How to Import These Workflow Templates

> All JSON files in this folder are ready-to-import n8n workflows.

---

## Method 1 — Import from File (Recommended)

```
1. Download the JSON file to your computer
2. In n8n: top-right corner → click the ☰ menu
3. Select: Import from File
4. Choose your downloaded JSON file
5. Click Import
6. Workflow opens in the editor
7. Replace all credential placeholders (see below)
8. Test with Manual Trigger first
9. Activate when ready
```

## Method 2 — Copy Paste

```
1. Open the JSON file on GitHub → click Raw
2. Copy all content (Ctrl+A, Ctrl+C)
3. In n8n: ☰ → Import from URL → paste raw GitHub URL
   OR: paste JSON content directly if the option is available
```

---

## After Import: Replace These Placeholders

Search each workflow for `REPLACE_WITH_` and update:

| Placeholder | What to Put |
|---|---|
| `REPLACE_WITH_YOUR_TELEGRAM_CREDENTIAL_ID` | Your Telegram credential ID from Settings → Credentials |
| `REPLACE_WITH_YOUR_TELEGRAM_CHAT_ID` | Your Chat ID (get from https://api.telegram.org/bot{TOKEN}/getUpdates) |
| `REPLACE_WITH_YOUR_GOOGLE_CREDENTIAL_ID` | Your Google OAuth2 credential ID |
| `REPLACE_WITH_YOUR_SPREADSHEET_ID` | Your Google Sheet ID (from URL) |
| `REPLACE_WITH_YOUR_GITHUB_CREDENTIAL_ID` | Your GitHub credential ID |
| `REPLACE_WITH_YOUR_GITHUB_USERNAME` | Your GitHub username (e.g., Shri-Phnx) |
| `REPLACE_WITH_N8N_API_KEY_CREDENTIAL_ID` | Header Auth credential with n8n API key |

---

## Available Templates

| File | Workflow | Trigger | Key Nodes |
|---|---|---|---|
| `01-it-telegram-ticket-bot.json` | IT Support Ticket Bot | Telegram message | If → Sheets → Telegram |
| `02-daily-morning-briefing.json` | Daily 9 AM Status Report | Schedule (9 AM) | Sheets → Code → Telegram |
| `03-n8n-backup-to-github.json` | Auto-backup all workflows | Schedule (2 AM) | n8n API → GitHub |
| `04-careerforge-lead-qualifier.json` | Score and route leads | Webhook | Code (score) → Switch → Telegram |
| `05-gmail-application-tracker.json` | Track job applications | Gmail Trigger | Code → Sheets → Telegram |

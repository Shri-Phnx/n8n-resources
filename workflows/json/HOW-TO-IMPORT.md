# How to Import n8n Workflow JSON Files

> **Author:** Shrinivas Ramaprasad | **Updated:** May 2026

---

## Step-by-Step Import

```
Step 1: Open n8n in your browser (http://localhost:5678 or your domain)

Step 2: Click the menu icon (☰) in the top-right corner of the editor
        OR go to the main Workflows list

Step 3: Click Import from File
        OR use keyboard shortcut: Ctrl+Shift+I (Windows) / Cmd+Shift+I (Mac)

Step 4: Navigate to the JSON file you downloaded from this folder
        Select the file → click Open / Import

Step 5: The workflow loads in the editor
        All nodes appear on the canvas with connections

Step 6: Connect your credentials
        Each node that needs a credential shows a red warning
        Click the node → Credential dropdown → select your saved credential
        (The node stores the credential NAME, not the actual secret)

Step 7: Update hard-coded values
        Search for REPLACE_WITH_ in node parameters
        Update Chat IDs, Spreadsheet URLs, Channel names etc.

Step 8: Click Execute Workflow to test with Manual Trigger
        OR activate the workflow using the toggle in the header
```

---

## What Gets Preserved on Import

| Preserved | NOT Preserved |
|---|---|
| All nodes and their settings | Credential secrets |
| All connections between nodes | Credential IDs (must re-link) |
| Node positions on canvas | Execution history |
| Workflow name | Pinned test data |
| Node notes | Workflow active/inactive state |

---

## After Import — Checklist

```
1. Re-link all credentials (red warning icons on nodes)
2. Update REPLACE_WITH_CHAT_ID with your Telegram Chat ID
3. Update REPLACE_WITH_SPREADSHEET_URL with your Google Sheets URL
4. Update REPLACE_WITH_CHANNEL with your Slack channel name
5. Test with Execute Workflow before activating
6. Check execution logs for any errors
```

---

## Files in This Folder

| File | Workflow | Nodes Used |
|---|---|---|
| `01-it-telegram-ticket-bot.json` | Telegram /ticket command → Sheets + Slack | Telegram Trigger, If, Set, Sheets, Slack, Telegram |
| `02-daily-morning-briefing.json` | 9 AM schedule → Sheets → Telegram + Email | Schedule, Sheets, Code, Telegram, Gmail |
| `03-n8n-backup-to-github.json` | Daily 2 AM → backup all workflows → GitHub | Schedule, HTTP Request, Split Out, Loop, Code, GitHub |
| `04-careerforge-lead-qualifier.json` | Webhook → score → HubSpot + Telegram | Webhook, Set, Code, Switch, Telegram, Gmail, Sheets |
| `05-gmail-application-tracker.json` | Gmail → classify → Sheets upsert | Gmail Trigger, Set, Code, Sheets, Gmail (label), Telegram |

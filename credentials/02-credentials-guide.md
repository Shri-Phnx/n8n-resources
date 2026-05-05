# n8n Credentials Guide — Part 2: Messaging, Dev Tools & Databases

> Author: Shrinivas Ramaprasad | May 2026

---

## Telegram Bot Token

```
Step 1 — Create bot:
1. Telegram → search @BotFather → send /newbot
2. Display name: My n8n Bot
3. Username: my_n8n_automation_bot  (must end in 'bot', globally unique)
4. Token: 7123456789:AAFxyz-AbCdEfGhIjKlMnOpQrStUvWxYz
   ⚠️ SAVE IMMEDIATELY

Step 2 — Configure in n8n:
  Settings → Credentials → + Add → Telegram
  Credential Name: Telegram - IT Support Bot
  Access Token: paste → Save → ✅ Connected

Step 3 — Get your Chat ID:
1. Send /start to your bot in Telegram
2. Visit: https://api.telegram.org/bot{TOKEN}/getUpdates
3. Find: result[0].message.chat.id
   Positive = personal | Negative = group | -100xxx = channel
```

| Field | Sample Value | Validation |
|---|---|---|
| Credential Name | `Telegram - IT Support Bot` | Any name |
| Access Token | `7123456789:AAFxyz-AbCde...` | numbers:alphanumeric |

**⚠️ Common Telegram mistakes:**

| Mistake | Effect | Fix |
|---|---|---|
| User never sent /start | Cannot message user | User must initiate first |
| Wrong Chat ID sign | Wrong recipient | Groups: negative; personal: positive |
| Bot not added to group | Can't post | Add bot as group member |
| Token has spaces | Auth fails | Trim whitespace |

---

## Slack Bot Token

```
Step 1 — Create Slack App:
  api.slack.com/apps → Create New App → From scratch
  Name: n8n Automation | Workspace: yours → Create

Step 2 — Add scopes (OAuth & Permissions → Bot Token Scopes):
  chat:write, channels:read, channels:history,
  users:read, files:write, app_mentions:read, im:write

Step 3 — Install to Workspace → Allow
  Copy: Bot User OAuth Token (starts with xoxb-)

Step 4 — Configure in n8n:
  Credentials → + Add → Slack
  Access Token: xoxb-... → Save → ✅ Connected

Step 5 — Invite bot to channel (REQUIRED):
  /invite @YourBotName
```

| Field | Sample Value | Where to Get |
|---|---|---|
| Access Token | `xoxb-1234567890-1234567890123-...` | Slack App → OAuth & Permissions |

**⚠️ Common Slack mistakes:**

| Mistake | Error | Fix |
|---|---|---|
| Bot not in channel | `not_in_channel` | /invite @YourBotName |
| Missing scope | `missing_scope` | Add scope → reinstall app |
| Using xoxp- (user token) | Wrong permissions | Use xoxb- (Bot token) |

---

## GitHub Personal Access Token

```
1. github.com → Settings → Developer settings
   → Fine-grained tokens → Generate new token
   Name: n8n-automation | Expiration: 90 days
   Permissions:
     Contents: Read and Write
     Metadata: Read-only (always required)
     Issues: Read and Write
     Pull requests: Read and Write
   Token starts with: github_pat_11...

2. n8n: Credentials → + Add → GitHub
   Access Token: github_pat_... → Save
```

---

## SMTP Email

| Field | Gmail | Outlook |
|---|---|---|
| Host | `smtp.gmail.com` | `smtp.office365.com` |
| Port | `587` | `587` |
| Security | STARTTLS | STARTTLS |
| User | your@gmail.com | your@company.com |
| Password | App Password (16 chars) | Your password |

**Gmail App Password:**

```
1. myaccount.google.com → Security
2. Enable 2-Step Verification
3. Search "App passwords" → App: Mail → Other: n8n → Generate
4. Copy 16-char password → use in n8n (NOT your Gmail password)
```

---

## MySQL Credential

| Field | Sample Value | If Wrong |
|---|---|---|
| Host | `localhost` or `192.168.1.100` | Connection refused |
| Database | `crm_db` | Unknown database |
| User | `n8n_user` | Access denied |
| Password | `SecurePass123!` | Access denied |
| Port | `3306` | Connection refused |
| SSL | ON for remote/cloud | Data unencrypted |

```sql
-- Create dedicated MySQL user
mysql -u root -p
CREATE USER 'n8n_user'@'%' IDENTIFIED BY 'SecurePass123!';
GRANT SELECT, INSERT, UPDATE, DELETE ON crm_db.* TO 'n8n_user'@'%';
FLUSH PRIVILEGES;
```

---

## PostgreSQL Credential

| Field | Sample Value | If Wrong |
|---|---|---|
| Host | `localhost` or `db.example.com` | Connection refused |
| Database | `crm` | DB does not exist |
| User | `n8n_user` | Authentication failed |
| Password | `SecurePass123!` | Authentication failed |
| Port | `5432` | Connection refused |
| SSL | ON for remote/cloud | Data unencrypted |
| Schema | `public` | Schema not found |

```sql
-- Create dedicated PostgreSQL user
sudo -u postgres psql
CREATE USER n8n_user WITH PASSWORD 'SecurePass123!';
GRANT CONNECT ON DATABASE crm TO n8n_user;
\c crm
GRANT USAGE ON SCHEMA public TO n8n_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO n8n_user;
```

---

## HubSpot Private App Token

```
1. HubSpot → Settings → Integrations → Private Apps → Create
   Name: n8n Automation
   Scopes: crm.objects.contacts.read/write, crm.objects.deals.read/write
2. Create app → copy Access Token (pat-na1-...)
3. n8n: Credentials → + Add → HubSpot → paste → Save
```

---

## Master Credential Checklist

```
✅ Always:
  Name: ServiceName - Purpose - Environment
  One credential per service per environment
  Minimum required permissions only
  Save to password manager immediately after creation
  Back up N8N_ENCRYPTION_KEY — losing it = permanent credential loss
  Test after creating (Test button)
  Rotate every 90 days for production

❌ Never:
  Hardcode keys in expressions, URLs, or node fields
  Export workflows with credentials included
  Use root/admin DB users for n8n connections
  Share credentials across environments
  Commit .env files with secrets to Git
```

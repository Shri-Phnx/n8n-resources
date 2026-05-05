# n8n Credentials Guide — Part 1: API Keys & OAuth2

> Author: Shrinivas Ramaprasad | May 2026

---

## How Credentials Work

Credentials are encrypted with `N8N_ENCRYPTION_KEY`. Decrypted only in memory during execution.

**Access:** Settings → Credentials → + Add Credential

**Naming rule:** `ServiceName - Purpose - Environment`
- ✅ `OpenAI - CareerForge Prod`
- ✅ `Gmail - Work Automated`
- ❌ `test` / `my cred 1`

**Critical:** Back up `N8N_ENCRYPTION_KEY` in a password manager. Losing it = losing all credentials permanently.

---

## Type 1 — API Key

Used by: OpenAI, Anthropic, Groq, NVIDIA NIM, Hugging Face.

| Field | Sample Value | Where to Get |
|---|---|---|
| Credential Name | `OpenAI - CareerForge Prod` | You choose |
| API Key | `sk-proj-abc123...` | Service developer dashboard → API Keys |

### OpenAI

```
1. platform.openai.com → Profile → API keys → + Create new secret key
   Name: n8n-automation
   ⚠️ Copy immediately — shown only once. Starts with sk-proj-
2. n8n: Settings → Credentials → + Add → OpenAI
   API Key: paste → Save → ✅ Connected
```

### Anthropic (Claude)

```
1. console.anthropic.com → Settings → API Keys → + Create Key
   Name: n8n | Copy (starts with sk-ant-api03-)
2. n8n: Credentials → + Add → Anthropic
   API Key: paste → Save
```

### Groq

```
1. console.groq.com → API Keys → Create API Key | Copy (starts with gsk_)
2. n8n: Credentials → + Add → Groq | API Key: paste → Save
```

### NVIDIA NIM

```
1. build.nvidia.com → Profile → Generate API Key | Copy (nvapi-)
2. n8n: HTTP Request node → Header Auth credential
   Header Name: Authorization
   Header Value: Bearer nvapi-your-key
```

**✅ Best practices:**
- Separate key per application and per environment
- Save to password manager before closing that tab
- Rotate every 90 days for production
- Use scoped/restricted keys where the service supports it

**⚠️ Common mistakes:**

| Mistake | Effect | Fix |
|---|---|---|
| Sharing key across environments | Revoking dev breaks prod | Separate keys per env |
| Not saving key immediately | Must regenerate | Password manager first |
| Spaces when pasting | 401 Unauthorized | Trim whitespace |
| Hardcoding key in URL/expression | Exposed in exports | Always use credential reference |

---

## Type 2 — Google OAuth2

### Option A: Quick Sign-In (Personal)

```
1. n8n: Settings → Credentials → + Add → Gmail OAuth2 API
   (same credential for Sheets, Drive, Calendar, Docs)
2. Credential Name: Google - Shrinivas Personal
3. Click Sign in with Google → select account → Allow
4. Status: ✅ Connected (token auto-refreshes)
```

### Option B: Cloud Console (Workspace / Custom App)

**Step 1 — Enable APIs:**

```
1. console.cloud.google.com → New Project: n8n-automation
2. APIs & Services → Library → Enable each:
   Gmail API, Google Sheets API, Google Drive API,
   Google Calendar API, Google Docs API
```

**Step 2 — OAuth Consent Screen:**

```
1. APIs & Services → OAuth consent screen
2. User Type: External (personal) or Internal (Workspace)
3. App name: n8n Automation | Support email: your@gmail.com
4. Test users: add your Gmail address → Save
```

**Step 3 — Create OAuth Client:**

```
1. Credentials → + Create Credentials → OAuth client ID
2. Application type: Web application | Name: n8n Client
3. Authorized redirect URIs (EXACT — no trailing slash):
   https://your-n8n.com/rest/oauth2-credential/callback
4. Create → Copy Client ID and Client Secret
```

**Step 4 — Configure in n8n:**

| Field | Sample Value | Source |
|---|---|---|
| Client ID | `123456789-abc.apps.googleusercontent.com` | Google Cloud → Credentials |
| Client Secret | `GOCSPX-abc123...` | Google Cloud → Credentials |

```
→ Click Sign in with Google → authorize → ✅ Connected
```

**⚠️ Common OAuth2 mistakes:**

| Mistake | Error | Fix |
|---|---|---|
| Trailing slash in redirect URI | `redirect_uri_mismatch` | Remove `/` at end |
| Using http:// in production | OAuth rejected | Must use https:// |
| API not enabled in Console | `403 accessNotConfigured` | Enable API in Library |
| Token expired (6+ months inactive) | `401 invalid_grant` | Re-authenticate credential |
| User not in test list | `403 access_blocked` | Add email to test users |

---

## Type 3 — Google Service Account

Use when: automating Workspace without human OAuth popup.

```
Step 1:
1. console.cloud.google.com → IAM & Admin → Service Accounts
2. + Create: Name: n8n-automation | Role: Editor
3. Keys tab → Add Key → JSON → Create → Download JSON file

Step 2 — Domain-Wide Delegation:
1. admin.google.com → Security → API Controls → Domain-wide Delegation
2. Add new:
   Client ID: (from JSON → "client_id" field)
   Scopes:
     https://mail.google.com/,
     https://www.googleapis.com/auth/spreadsheets,
     https://www.googleapis.com/auth/drive

Step 3 — Configure in n8n:
  Settings → Credentials → + Add → Google Service Account
```

| Field | Sample | Source |
|---|---|---|
| Service Account Email | `n8n-auto@project.iam.gserviceaccount.com` | JSON → `client_email` |
| Private Key | `-----BEGIN PRIVATE KEY-----\nMIIE...` | JSON → `private_key` (full block) |
| Impersonate User | `shrinivas@yourcompany.com` | Workspace email |

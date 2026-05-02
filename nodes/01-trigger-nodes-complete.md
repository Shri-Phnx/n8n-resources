# n8n Trigger Nodes — Complete Step-by-Step Guide

> **Author:** Shrinivas Ramaprasad | **Last updated:** April 2026
> Every trigger type, every field, every option — with implementation steps, execution steps, requirements, tips, and common mistakes.

---

## What Is a Trigger Node?

A trigger node is the **first node in every workflow**. It determines:
- **When** the workflow starts
- **What data** enters the workflow at the beginning
- **How** the workflow is activated (event, schedule, manual, webhook, etc.)

Without a trigger node, a workflow cannot run.

**Rule:** Every workflow must have exactly **one** trigger node as its starting point.

---

## All Trigger Types in n8n

When you click the `+` button on an empty canvas, n8n shows you all ways to start a workflow:

| Display Name | Underlying Node | When It Fires |
|---|---|---|
| **Trigger manually** | Manual Trigger | You click the button |
| **On App events** | App-specific triggers (Gmail, Slack, Airtable, etc.) | When an event happens in a connected app |
| **On a schedule** | Schedule Trigger | At a time/interval you define |
| **On webhook call** | Webhook | When an external HTTP request arrives |
| **On form submission** | n8n Form Trigger | When someone submits an n8n-hosted form |
| **When called by another workflow** | Execute Sub-workflow Trigger | When a parent workflow calls this one |
| **On chat message** | Chat Trigger | When a user sends a message in n8n's chat UI |
| **Other ways** | Error Trigger, Activation Trigger, SSE Trigger, etc. | Specific platform events |

---

## 1. Trigger Manually (Manual Trigger)

### What It Is
The simplest trigger. You run the workflow by clicking the **"Execute workflow"** button in the n8n editor. No automation — purely manual.

### Requirements
- None. Zero configuration. Works out of the box.

### When to Use
- Testing new workflows before adding a real trigger
- One-time data migrations (import CSV, clean database, etc.)
- Running workflows ad-hoc on demand
- Learning n8n — always start with this before scheduling

### Fields (Parameters Tab)
This node has **no configurable fields**. It outputs one empty item `{}` when clicked.

### Fields (Settings Tab)
| Field | Value | What It Does |
|---|---|---|
| Always Output Data | OFF (default) | Node passes data even if empty |
| Execute Once | OFF | Run for every item (not applicable here — there's only 1 item) |
| Retry On Fail | OFF | No retries needed for trigger |
| On Error | Stop Workflow | What to do if this node errors |
| Notes | (empty) | Add notes for your team |

### Implementation — Step by Step
```
1. Open n8n editor → click New Workflow
2. Click the + icon on the canvas (or press N)
3. Under "How should this workflow be triggered?"
4. Click "Trigger manually"
5. The Manual Trigger node appears on canvas
6. Build your workflow by adding nodes after it
7. Click the orange "Execute workflow" button to run
```

### Execution — What Happens
```
Click "Execute workflow"
    → Manual Trigger outputs: [{ json: {} }]
    → n8n passes this empty item to the next node
    → Downstream nodes run in sequence
    → Results appear in each node's output panel
```

### Tips
- ✅ Always use this trigger when building and testing — avoids accidental real triggers
- ✅ You can add "Test data" to the Manual Trigger using the **"Edit output"** option (right-click node → Edit output) to simulate real data without calling external APIs
- ✅ After testing, swap to Schedule or Webhook trigger without changing any other node

### Common Mistakes
| Mistake | Effect | Fix |
|---|---|---|
| Activating a workflow that only has Manual Trigger | Workflow never auto-runs | Only webhook/schedule workflows need activation. Manual trigger workflows run when you click the button |
| Forgetting to remove Manual Trigger before production | Workflow doesn't auto-run | Replace with Schedule or Webhook trigger for automation |

---

## 2. On App Events (App-Specific Trigger Nodes)

### What It Is
Triggered by events in external applications (Gmail, Slack, GitHub, Shopify, Airtable, Notion, etc.). n8n polls the app or receives a webhook from it and fires when the defined event occurs.

### Available App Triggers (Key Examples)
| App | Trigger Events | Method |
|---|---|---|
| **Gmail Trigger** | New email, new label, new thread | Polling (every minute) |
| **Slack Trigger** | Message posted, reaction added, new channel | Webhook (real-time) |
| **GitHub Trigger** | Push, PR, issue, release, star | Webhook (real-time) |
| **Google Sheets Trigger** | Row added or updated | Polling |
| **Airtable Trigger** | Record created, updated, deleted | Polling |
| **Notion Trigger** | Database item created/updated | Polling |
| **HubSpot Trigger** | Contact/deal created, property changed | Webhook |
| **Stripe Trigger** | Payment, subscription, customer events | Webhook |
| **Shopify Trigger** | Order, product, customer events | Webhook |
| **Jira Trigger** | Issue created, updated, status changed | Webhook |
| **Telegram Trigger** | Bot message received | Polling or Webhook |
| **Discord Trigger** | Message in channel | Webhook |
| **Typeform Trigger** | Form submitted | Webhook |
| **Postgres Trigger** | Database changes (LISTEN/NOTIFY) | Real-time |
| **MySQL** | No native trigger — use Schedule + MySQL node | Polling |

### Deep Dive: Gmail Trigger

#### Credential Required
- Type: **Google (OAuth2)**
- Scopes needed: `https://mail.google.com/` or `https://www.googleapis.com/auth/gmail.readonly`

#### How to Get Gmail Credentials
```
Option A — OAuth2 (recommended for personal Gmail):
1. In n8n: Settings → Credentials → Add → search "Gmail"
2. Click "Sign in with Google"
3. Google OAuth popup opens → select your Gmail account
4. Grant permissions → credential is saved automatically

Option B — Service Account (for Google Workspace):
1. Go to console.cloud.google.com
2. Create a project → Enable Gmail API
3. IAM & Admin → Service Accounts → Create Service Account
4. Download JSON key file
5. In n8n: Gmail credential → select "Service Account"
   → paste service account email
   → paste private key from JSON file
```

#### Gmail Trigger Fields — Field by Field

| Field | Options | Sample Value | What It Does | If Wrong/Empty |
|---|---|---|---|---|
| **Credential** | Dropdown of saved Gmail creds | `Gmail - Shrinivas Work` | Authenticates the connection | ❌ Node fails — credential required |
| **Resource** | (fixed) Messages | — | What to watch | — |
| **Trigger On** | New Email, New Thread, New Label Added | `New Email` | Which Gmail event fires the trigger | Wrong event = trigger never fires or fires too often |
| **Label** | Dropdown of your Gmail labels | `INBOX`, `STARRED`, `work/leads` | Only trigger on emails with this label | If empty: watches ALL emails including spam |
| **Read Status** | Unread only, Read only, Any | `Unread only` | Filter by read/unread status | `Any` can cause duplicate processing of already-read emails |
| **Sender** (Optional) | Text field | `noreply@stripe.com` | Only trigger on emails from this sender | If empty: all senders |
| **Subject Contains** (Optional) | Text field | `Invoice #` | Only trigger if subject contains this text | Case-insensitive match |
| **Poll Times** | Every N Minutes | `1` (minute) | How often n8n checks Gmail for new emails | Higher = less real-time but less API quota usage |

#### Gmail Trigger Output
```json
{
  "id": "18abc123def456",
  "threadId": "18abc123def456",
  "snippet": "Hi Shrinivas, your invoice is attached...",
  "from": "billing@company.com",
  "to": "shrinivas@example.com",
  "subject": "Invoice #INV-2026-001",
  "date": "2026-04-30T09:15:00.000Z",
  "body": {
    "text": "Hi Shrinivas, please find your invoice attached.",
    "html": "<p>Hi Shrinivas, please find your invoice attached.</p>"
  },
  "attachments": [
    {
      "filename": "invoice-001.pdf",
      "mimeType": "application/pdf",
      "size": 45231
    }
  ],
  "labels": ["INBOX", "UNREAD"]
}
```

#### Gmail Trigger — Implementation Steps
```
Step 1: Set up credential
  → Settings → Credentials → Add → Gmail OAuth2
  → Click "Sign in with Google" → authorize

Step 2: Add Gmail Trigger to canvas
  → Press N → search "Gmail" → select "Gmail Trigger"

Step 3: Configure the trigger
  → Credential: select your Gmail credential
  → Trigger On: "New Email"
  → Label: INBOX
  → Read Status: Unread only
  → Poll Times: Every 1 Minute

Step 4: Test
  → Click "Listen for test event"
  → Send a test email to your Gmail account
  → n8n shows the captured payload within 1 minute

Step 5: Activate
  → Toggle "Active" switch at top of editor
  → Workflow now runs automatically every 1 minute
  → It only processes emails it hasn't seen before (n8n tracks this internally)
```

#### Gmail Trigger — Tips & Common Mistakes
| Issue | Cause | Fix |
|---|---|---|
| Trigger fires for old emails | `Read Status = Any` and all old emails are unread | Set `Read Status = Unread only`; mark old emails as read in Gmail |
| Same email processed twice | Workflow ran while email was partially processed | n8n uses the email ID to track — check if execution succeeded |
| Trigger not firing | Workflow not activated | Toggle the Active switch in the editor |
| OAuth token expired | Google revokes tokens after 6 months of inactivity | Re-authenticate the credential in Settings → Credentials |
| Missing attachment content | n8n captures attachment metadata, not content by default | Use Gmail node (not trigger) with "Download Attachments" option in the next node |

---

### Deep Dive: Slack Trigger

#### Credential Required
- Type: **Slack OAuth2 API** or **Slack Bot Token**
- Required scopes: `channels:read`, `chat:write`, `reactions:read`, `users:read`, `app_mentions:read`

#### How to Get Slack Credentials
```
1. Go to: api.slack.com/apps
2. Click "Create New App" → "From scratch"
3. Name: "n8n Automation", Workspace: select yours → Create App
4. Left sidebar → OAuth & Permissions
5. Under "Bot Token Scopes", add:
   - channels:history (read messages)
   - chat:write (send messages)
   - channels:read (list channels)
   - reactions:read (read reactions)
   - users:read (read user info)
6. Scroll up → Click "Install to Workspace" → Authorize
7. Copy the "Bot User OAuth Token" (starts with xoxb-)
8. In n8n: Settings → Credentials → Add → Slack
   → Token: paste xoxb-your-token
   → Save
9. Enable Event Subscriptions:
   → api.slack.com/apps → your app → Event Subscriptions → Enable
   → Request URL: paste n8n webhook URL (from n8n Slack Trigger node)
   → Subscribe to bot events: message.channels, reaction_added, app_mention
10. Reinstall app to workspace after adding events
```

#### Slack Trigger Fields
| Field | Options | Sample | What It Does |
|---|---|---|---|
| **Credential** | Dropdown | `Slack - n8n Bot` | Auth connection |
| **Events to Listen For** | Message Posted, Reaction Added, New Channel, New Member, App Mention, Channel Archived | `Message Posted` | Which Slack event fires the trigger |
| **Watch Specific Channel** | Toggle | ON | Limit to one channel |
| **Channel** | Dropdown/text | `#support-tickets` | Which channel to watch |

#### Slack Trigger Output (Message Posted)
```json
{
  "type": "message",
  "text": "@n8n-bot please create a ticket for login issue",
  "user": "U01ABC123",
  "username": "shrinivas.r",
  "channel": "C01DEF456",
  "channel_name": "support-tickets",
  "ts": "1714456800.000000",
  "team": "T01GHI789",
  "blocks": [...]
}
```

---

## 3. On a Schedule (Schedule Trigger)

### What It Is
Runs the workflow automatically at a defined time or interval — like a cron job.

### Requirements
- No credentials needed
- n8n must be **running** (the workflow must be **Active**)
- Server timezone must be configured correctly

### Schedule Trigger Fields — Field by Field

#### Field: Trigger Interval
| Option | Sample Config | When It Fires | Best For |
|---|---|---|---|
| **Every N Minutes** | Every `15` Minutes | 12:00, 12:15, 12:30... | API polling, monitoring |
| **Every N Hours** | Every `2` Hours | 00:00, 02:00, 04:00... | Hourly sync, reports |
| **Every Day** | At `09:00` | Every day at 9 AM | Daily reports, summaries |
| **Every Week** | Monday at `09:00` | Every Monday 9 AM | Weekly digests |
| **Every Month** | 1st at `00:00` | 1st of each month at midnight | Monthly billing, cleanup |
| **Custom (Cron)** | `0 9 * * 1-5` | Weekdays at 9 AM | Complex schedules |

#### Field: Hour / Minute (for Day/Week/Month modes)
| Field | Range | Sample | Effect |
|---|---|---|---|
| Hour | 0–23 | `9` | Fires at 9:xx |
| Minute | 0–59 | `30` | Fires at x:30 |
| Day of Week | Monday–Sunday | `Monday` | Which day |
| Day of Month | 1–31 | `1` | First of month |

#### Field: Cron Expression (Custom mode)
```
Format: second  minute  hour  day-of-month  month  day-of-week
Short:  (omit second)  minute  hour  day  month  weekday

Examples:
  0 9 * * *       → Every day at 9:00 AM
  0 9 * * 1-5     → Weekdays (Mon-Fri) at 9 AM
  0 */2 * * *     → Every 2 hours
  30 8 * * 1      → Every Monday at 8:30 AM
  0 0 1 * *       → 1st of every month at midnight
  0 9,17 * * *    → Twice a day: 9 AM and 5 PM
  */15 * * * *    → Every 15 minutes
  0 0 * * 0       → Every Sunday at midnight

Validator: https://crontab.guru
```

#### Field: Timezone
| What It Is | The timezone for interpreting the schedule |
|---|---|
| **Sample value** | `Asia/Kolkata` (IST), `Asia/Dubai` (UAE), `Europe/London`, `America/New_York` |
| **Default** | Uses `GENERIC_TIMEZONE` env variable (often UTC) |
| **⚠️ Critical** | If your n8n server is in UTC and you don't set timezone, `09:00` fires at 9 AM UTC = 2:30 PM IST. ALWAYS set this |
| **Where to set globally** | Set `GENERIC_TIMEZONE=Asia/Kolkata` in your .env file (affects ALL workflows) |
| **Where to set per-workflow** | Schedule Trigger node → Settings tab → Timezone dropdown |

### Implementation — Step by Step
```
Step 1: Add Schedule Trigger
  → Press N → search "Schedule" → click Schedule Trigger

Step 2: Set the interval
  → Trigger Interval: "Every Day"
  → Hour: 9
  → Minute: 0

Step 3: Set timezone
  → Click "Add Option" → select "Timezone"
  → Type: Asia/Kolkata

Step 4: Build rest of workflow
  → Add processing nodes after the trigger

Step 5: Test
  → Click "Execute workflow" to test right now (manual test)
  → Verify data flows correctly through all nodes

Step 6: Activate
  → Toggle the Active switch in the editor header
  → Status shows "Active"
  → n8n will fire at 9:00 AM IST every day

Step 7: Verify it ran
  → Executions tab shows history of all runs
  → Check for green (success) or red (error)
```

### Schedule Trigger Output
```json
{
  "timestamp": "2026-04-30T09:00:00.000+05:30",
  "workflow": {
    "id": "wf_abc123",
    "name": "Daily Report Generator"
  }
}
```

### Tips & Common Mistakes
| Issue | Cause | Fix |
|---|---|---|
| Schedule doesn't fire | Workflow not Active | Toggle Active switch |
| Fires at wrong time | Timezone not set | Set GENERIC_TIMEZONE in .env AND in the Schedule node |
| Fires twice | Two n8n instances running | Ensure only one n8n process is running |
| Skipped a run | n8n was down during scheduled time | n8n does NOT catch up missed runs — the next scheduled time fires next |
| Too many API calls | Very short interval | Check API rate limits; use webhooks instead of polling where possible |

---

## 4. On Webhook Call (Webhook Trigger)

> See `02-webhook-deep-dive.md` for the comprehensive guide. Summary here.

### What It Is
n8n creates an HTTP endpoint. When any external service POSTs/GETs to that URL, the workflow fires.

### Requirements
- n8n must be accessible from the internet (or the calling service's network)
- WEBHOOK_URL environment variable must be set correctly
- Workflow must be **Active** for the production URL to work

### Fields — Field by Field

| Field | Options | Sample Value | What It Does | If Wrong/Empty |
|---|---|---|---|---|
| **HTTP Method** | GET, POST, PUT, PATCH, DELETE, HEAD | `POST` | Which HTTP method to accept | If you set GET but caller sends POST: 404 |
| **Path** | Any text, auto-generated UUID | `stripe-payment` | The URL path suffix | If empty: random UUID generated |
| **Authentication** | None, Basic Auth, Header Auth, JWT | `Header Auth` | How to verify callers | None = anyone can trigger your workflow |
| **Respond** | Immediately, Last Node, Respond to Webhook Node | `Immediately` | When/what to send back | Immediately = fastest, no response data |
| **Response Code** | Any HTTP status (200, 201, 400...) | `200` | HTTP status sent to caller | Wrong code may confuse calling service |
| **Response Data** | All Entries, First Entry, No Data | `No Data` | What body to include in response | Only relevant when Respond = Last Node |

### Webhook Trigger Output
```json
{
  "headers": {
    "content-type": "application/json",
    "x-api-key": "your-secret",
    "user-agent": "Stripe/1.0"
  },
  "params": {},
  "query": {
    "source": "stripe"
  },
  "body": {
    "id": "evt_123",
    "type": "payment_intent.succeeded",
    "data": {
      "object": {
        "amount": 5000,
        "currency": "usd",
        "customer": "cus_abc"
      }
    }
  }
}
```

### Access Incoming Data
```
Body field:   {{ $json.body.fieldName }}
Header:       {{ $json.headers['x-api-key'] }}
Query param:  {{ $json.query.paramName }}
Path param:   {{ $json.params.id }}
```

### Implementation — Step by Step
```
Step 1: Add Webhook Trigger
  → Press N → "On webhook call" → select Webhook

Step 2: Configure
  → HTTP Method: POST
  → Path: my-integration  (creates URL: .../webhook/my-integration)
  → Authentication: Header Auth
  → Create new Header Auth credential:
    → Header Name: X-Webhook-Secret
    → Header Value: (generate with: openssl rand -hex 32)

Step 3: Copy the test URL
  → URL shown: https://your-n8n.com/webhook-test/my-integration
  → Use THIS URL for testing in Postman or curl

Step 4: Test the webhook
  → Click "Listen for test event" in the node panel
  → In Postman: POST to the test URL
  → Body: { "test": true, "name": "Shrinivas" }
  → Headers: X-Webhook-Secret: your-generated-secret
  → n8n shows the captured payload

Step 5: Build rest of workflow
  → Access body: {{ $json.body.name }}

Step 6: Activate for production
  → Toggle Active switch
  → Production URL: .../webhook/my-integration (no -test)
  → Use this URL in the external service
```

---

## 5. On Form Submission (n8n Form Trigger)

### What It Is
n8n hosts a web form at a unique URL. When someone submits the form, the workflow fires with the form data.

### Requirements
- n8n must be accessible from the internet (for external users to see the form)
- WEBHOOK_URL must be set to your public domain
- No external credentials needed — n8n handles the form hosting

### Form Trigger Fields — Field by Field

| Field | Options | Sample | What It Does |
|---|---|---|---|
| **Form Title** | Any text | `IT Support Request` | Shown at top of the form page |
| **Form Description** | Any text | `Fill in your details and we'll respond within 24h` | Subtext shown below title |
| **Form Fields** | (add as many as needed) | — | Each field becomes a question on the form |
| **Respond To Form** | Using Respond Node, Immediately | `Using Respond Node` | Whether to show a custom response after submission |

#### Form Field Sub-Options
| Sub-field | Options | Sample | What It Does |
|---|---|---|---|
| **Label** | Text | `Your Name` | Question label shown to user |
| **Type** | Text, Email, Number, Date, Time, Password, Dropdown, Multi-Select, Textarea, File, Toggle | `Email` | Input type |
| **Required** | Toggle ON/OFF | ON | Whether user must fill this field |
| **Placeholder** | Text | `e.g. shrinivas@example.com` | Hint text inside the input |
| **Default Value** | Text | (empty usually) | Pre-filled value |
| **Field Name (key)** | Auto or custom | `email` | The JSON key in the output. Use lowercase no spaces |

#### For Dropdown / Multi-Select fields:
| Sub-field | Sample | What It Does |
|---|---|---|
| **Options** | `HR, IT, Finance, Operations` | Comma-separated list shown as dropdown options |
| **Allow Other** | Toggle | Lets user type a custom value not in the list |

### Form Trigger Output
```json
{
  "formMode": "test",
  "submittedAt": "2026-04-30T09:15:00.000Z",
  "data": {
    "name": "Shrinivas Ramaprasad",
    "email": "shrinivas@example.com",
    "department": "IT",
    "issue": "Cannot access VPN",
    "priority": "High"
  }
}
```

### Access Form Data
```
{{ $json.data.email }}
{{ $json.data.name }}
{{ $json.data.department }}
```

### Implementation — Step by Step
```
Step 1: Add Form Trigger
  → Press N → "On form submission" → n8n Form Trigger

Step 2: Set form details
  → Form Title: IT Support Request
  → Form Description: Submit your IT issue and we'll get back to you within 2 hours

Step 3: Add form fields
  → Add Field:
    Label: Full Name | Type: Text | Required: ON | Field Name: name
  → Add Field:
    Label: Email | Type: Email | Required: ON | Field Name: email
  → Add Field:
    Label: Department | Type: Dropdown | Required: ON
    Options: HR, IT, Finance, Operations | Field Name: department
  → Add Field:
    Label: Describe the Issue | Type: Textarea | Required: ON | Field Name: issue
  → Add Field:
    Label: Priority | Type: Dropdown
    Options: Low, Medium, High, Critical | Default: Medium | Field Name: priority

Step 4: Add Respond node (optional)
  → Add "Respond to Form" node after your processing
  → Set response message: "✅ Your request has been submitted! Ticket ID: {{ $json.ticketId }}"

Step 5: Test
  → Click "Listen for test event" in the node
  → n8n shows a "Open form" button → click it
  → Fill the form in your browser → Submit
  → n8n captures the data

Step 6: Activate
  → Toggle Active switch
  → Share the production form URL with your team
  → URL format: https://your-n8n.com/form/your-path
```

### Complete Use Case: IT Ticket Form → Slack + Google Sheets
```
n8n Form Trigger
    → Edit Fields (build ticket object)
        ticketId: TICKET-{{ $now.toMillis() }}
        status: Open
    → Google Sheets (Append Row — log ticket)
    → Slack (Send to #it-support channel)
        Message: 🎫 New ticket {{ $json.ticketId }}
                 From: {{ $json.data.name }} ({{ $json.data.department }})
                 Issue: {{ $json.data.issue }}
                 Priority: {{ $json.data.priority }}
    → Respond to Form
        Message: Your ticket {{ $json.ticketId }} has been created!
```

### Tips & Common Mistakes
| Issue | Cause | Fix |
|---|---|---|
| Form URL shows localhost | WEBHOOK_URL not set | Set WEBHOOK_URL=https://your-n8n.com in .env |
| File uploads not working | File size too large | Check n8n's max upload size (default 16 MB); increase via N8N_DEFAULT_BINARY_DATA_MODE |
| Test and production forms look different | Different field configs | They're the same — just different URLs |
| Dropdown shows wrong options | Typo in options list | Options are comma-separated — no space after comma |

---

## 6. When Called by Another Workflow (Execute Sub-workflow Trigger)

### What It Is
This workflow acts as a **sub-workflow** or **function** that another workflow calls. The calling workflow passes data in; this workflow processes it and returns a result.

### Requirements
- The calling workflow must use the **Execute Sub-workflow** node (not trigger)
- This trigger workflow must be **Active**
- Both workflows must be on the same n8n instance

### Execute Sub-workflow Trigger Fields
| Field | Options | Sample | What It Does |
|---|---|---|---|
| **No fields to configure** | — | — | This trigger simply exposes the workflow as callable. It outputs whatever data the calling workflow sends |

### How It Works — Architecture
```
Caller Workflow:
    Schedule Trigger
        → Get 100 contacts from CRM
        → Loop Over Items
              → Execute Sub-workflow node
                    Workflow: "Send Welcome Email" ← points to this workflow
                    Data: { name, email, plan }

This Workflow (Execute Sub-workflow Trigger):
    Execute Sub-workflow Trigger
        → Receives: { name: "Shrinivas", email: "s@x.com", plan: "Gold" }
        → Build personalized email
        → Send Email node
        → Return: { success: true, messageId: "msg_123" }
```

### Execute Sub-workflow Trigger Output
```json
// Whatever the calling workflow passes
{
  "name": "Shrinivas Ramaprasad",
  "email": "shrinivas@example.com",
  "plan": "Gold",
  "customerId": "cust_001"
}
```

### Implementation — Step by Step

#### Setting Up the Sub-workflow:
```
Step 1: Create a new workflow
  → New Workflow → name it clearly: "SUB: Send Welcome Email"

Step 2: Add Execute Sub-workflow Trigger
  → Press N → "When called by another workflow"
  → This node has no configuration

Step 3: Build the sub-workflow logic
  → Add processing nodes
  → Access incoming data: {{ $json.name }}, {{ $json.email }}

Step 4: Add a Return node (optional but recommended)
  → Add "Set" or "Edit Fields" node as the last node
  → Set: success = true, processedAt = {{ $now.toISO() }}
  → This data is returned to the calling workflow

Step 5: Activate the sub-workflow
  → Toggle Active switch
  → The sub-workflow is now callable
```

#### Setting Up the Calling Workflow:
```
Step 1: Add "Execute Sub-workflow" node (not trigger)
  → Press N → search "Execute Sub-workflow" (core node)

Step 2: Configure it
  → Source: Database
  → Workflow: select "SUB: Send Welcome Email" from dropdown
  → Input data: select "Using Fields Below" or "JSON"
  → Add fields to pass:
    name: {{ $json.contactName }}
    email: {{ $json.contactEmail }}
    plan: {{ $json.subscriptionPlan }}

Step 3: Wait for result
  → Toggle "Wait for Sub-workflow" ON to receive the return value
  → Access result: {{ $json.success }}, {{ $json.processedAt }}
```

### Why Use Sub-workflows?
| Use Case | Benefit |
|---|---|
| Send same notification from 10 different workflows | Define once, call everywhere |
| Complex transformation logic | Reusable function |
| Long-running tasks in parallel | Fire-and-forget async processing |
| Clean modular design | Easier to debug individual workflows |

### Tips & Common Mistakes
| Issue | Cause | Fix |
|---|---|---|
| "Workflow not found" error | Sub-workflow not active | Toggle Active in the sub-workflow |
| No data returned | No node after the trigger | Add processing nodes; last node's output is returned |
| Infinite loop | Sub-workflow calls itself | Never call the same workflow recursively |
| Data not passed | Field names case mismatch | Check exact field names; expressions are case-sensitive |

---

## 7. On Chat Message (Chat Trigger)

### What It Is
n8n hosts a chat interface (web UI or embeddable widget). When a user sends a message, the workflow fires — perfect for building AI chatbots and conversational agents.

### Requirements
- Workflow must be **Active**
- n8n must be accessible from the browser
- For AI responses: connect AI Agent node

### Chat Trigger Fields — Field by Field
| Field | Options | Sample | What It Does |
|---|---|---|---|
| **Authentication** | None, Basic Auth | `None` for internal, `Basic Auth` for external | Controls who can access the chat UI |
| **Allowed Origins (CORS)** | Text (URL) | `https://yourapp.com` | Which domains can embed the chat widget |
| **Initial Message** | Text | `Hi! I'm your IT support assistant. How can I help?` | First message shown when chat opens |
| **Show Welcome Screen** | Toggle | ON | Whether to show a welcome screen before chat |
| **Input Placeholder** | Text | `Type your IT issue here...` | Placeholder text in the chat input |
| **Load Previous Session** | Toggle | ON (recommended) | Show previous conversation when user returns |
| **Response Mode** | AI Response, Webhook Response | `AI Response` | How to handle user messages |

### Chat Trigger Output
```json
{
  "sessionId": "chat_abc123_shrinivas",
  "action": "sendMessage",
  "chatInput": "My laptop can't connect to VPN",
  "timestamp": "2026-04-30T09:15:00.000Z"
}
```

### Access Chat Data
```
User message:    {{ $json.chatInput }}
Session ID:      {{ $json.sessionId }}
```

### Implementation — Step by Step
```
Step 1: Add Chat Trigger
  → Press N → "On chat message" → Chat Trigger

Step 2: Configure
  → Initial Message: "Hi! I'm your IT assistant. What's your issue?"
  → Input Placeholder: "Describe your problem..."
  → Load Previous Session: ON
  → Authentication: None (for internal use)

Step 3: Add AI Agent
  → Connect AI Agent node
  → Connect Ollama Chat Model (or other LLM) to AI Agent
  → Connect Simple Memory (sessionId: {{ $json.sessionId }})
  → System prompt:
    "You are an IT support assistant.
    You help with VPN, laptop, software, and access issues.
    Always confirm the issue before suggesting a fix."

Step 4: Activate workflow

Step 5: Share the chat URL
  → n8n shows a "Chat" button in the editor
  → Production URL: https://your-n8n.com/chat/workflow-id
  → Share with your team

Step 6: Embed in a website (optional)
  → n8n provides a snippet like:
  <script src="https://your-n8n.com/webhook/chat-widget.js"></script>
  <n8n-chat webhook-url="https://your-n8n.com/webhook/chat-id"></n8n-chat>
```

### Complete AI Chat Workflow
```
Chat Trigger
    → AI Agent
          ├── [Model]   Ollama Chat Model (llama3.1:8b)
          ├── [Memory]  Redis Chat Memory
          │             Session: {{ $json.sessionId }}
          └── [Tools]
                ├── HTTP Request Tool → ServiceNow API
                └── Calculator
    → (AI Agent automatically sends response back to chat)
```

---

## 8. Other Trigger Modes

### Error Trigger
**What:** Fires when any workflow on the n8n instance encounters an error.
**Use:** Global error alerting — Slack notification, email, create ticket.

```
Setup:
1. Create a new workflow
2. Add Error Trigger node (no config)
3. Add Slack/Email notification
4. Activate
5. Set as global error handler:
   → Settings (gear icon) → n8n → Error Workflow → select this workflow
```

Output:
```json
{
  "workflow": { "id": "wf_123", "name": "Daily Report" },
  "execution": { "id": "ex_456", "mode": "trigger", "url": "https://n8n.com/exec/456" },
  "error": {
    "message": "HTTP Request failed: 401 Unauthorized",
    "stack": "Error at node 'Fetch CRM Data'..."
  },
  "trigger": { "type": "schedule" }
}
```

Useful expressions:
```
{{ $json.workflow.name }}      → Which workflow failed
{{ $json.error.message }}      → What went wrong
{{ $json.execution.url }}      → Direct link to failed execution
```

---

### Activation Trigger
**What:** Fires when the workflow itself is activated or deactivated.
**Use:** Run setup/teardown logic when enabling or disabling a workflow.

```json
// Output
{
  "event": "activate",    // or "deactivate"
  "workflowId": "wf_123",
  "workflowName": "Customer Sync",
  "timestamp": "2026-04-30T09:00:00Z"
}
```

Example use case:
```
Activation Trigger
    → If (event == 'activate')
          → Slack: "✅ Customer Sync workflow is now ACTIVE"
    → If (event == 'deactivate')
          → Slack: "⚠️ Customer Sync workflow was DEACTIVATED by {{ $json.user }}"
```

---

### n8n Form Trigger (Summary)
Already covered in section 5 above.

---

### Local File Trigger (Self-Hosted Only)
**What:** Fires when a file is created, modified, or deleted in a local folder.
**Use:** Watch a folder for new files, process them, move/archive after processing.

| Field | Sample | What It Does |
|---|---|---|
| Watch Path | `/home/n8n/incoming/` | Folder to monitor |
| Events | Created, Changed, Deleted | Which file events to react to |
| Ignore Files Starting With | `.` | Ignore temp/hidden files |
| Recursive | ON | Watch subfolders too |

```json
// Output
{
  "event": "created",
  "path": "/home/n8n/incoming/invoice-001.pdf",
  "type": "file",
  "stats": { "size": 45231 }
}
```

---

### MCP Server Trigger
**What:** Exposes the workflow as a tool that AI agents (like Claude) can call via the Model Context Protocol.
**Use:** Let external AI agents use your n8n workflow as a capability.

| Field | Sample | What It Does |
|---|---|---|
| Tool Name | `search_servicenow` | The name the AI agent uses to call this tool |
| Tool Description | `Search ServiceNow for open incidents` | Tells the AI when to use this tool |

---

## Quick Reference — All Triggers

| Trigger | Activation? | Credentials? | Real-time? | Best For |
|---|---|---|---|---|
| Manual | Not needed | None | On-demand | Testing |
| App Event | Yes | App credential | Varies | App integration |
| Schedule | Yes | None | No (polling) | Recurring jobs |
| Webhook | Yes | Optional | Yes | External events |
| Form | Yes | None | Yes | Data collection |
| Sub-workflow | Yes | None | Yes | Modular design |
| Chat | Yes | Optional | Yes | Chatbots |
| Error | Yes | None | Yes | Error alerting |
| Activation | Yes | None | Yes | Bootstrap/teardown |
| Local File | Yes | None | Yes | File processing |
| MCP Server | Yes | None | Yes | AI tool exposure |

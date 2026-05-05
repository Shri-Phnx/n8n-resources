# Pure Automation Workflows — No AI Required

> **Author:** Shrinivas Ramaprasad | **Updated:** May 2026
> All workflows here use only triggers, core nodes, and app integrations.
> AI is optional and always has a free/open-source fallback noted.

---

## Why Pure Automation First?

Before reaching for an AI agent, ask: can a simple rule or formula do this?

| Situation | Use AI? | Better approach |
|---|---|---|
| Route tickets by keyword | ❌ No | If / Switch node with text match |
| Score leads by title + years | ❌ No | Code node with numeric rules |
| Summarise a 50-page document | ✅ Yes | LLM is the right tool |
| Send a daily report | ❌ No | Schedule + Sheets + Code |
| Classify email as spam/lead | Maybe | Code rules first; AI if accuracy poor |

**Rule:** Pure automation is faster, cheaper, more predictable, and easier to debug.

---

## How to Use These Workflows

1. Read the node-by-node explanation below
2. Import the JSON from `workflows/json/` (see `HOW-TO-IMPORT.md`)
3. Replace credential placeholders with your saved credentials
4. Test with Execute Workflow (Manual Trigger mode)
5. Activate when confirmed working

---

## Workflow 1 — IT Telegram Ticket Bot

**Import file:** `workflows/json/01-it-telegram-ticket-bot.json`

### What it does

Users send `/ticket <describe issue>` to your Telegram bot. The workflow:
1. Detects the `/ticket` command
2. Creates a ticket record with ID, user, issue, timestamp
3. Logs to Google Sheets
4. Notifies `#it-support` on Slack
5. Replies to user with ticket ID and expected response time
6. For unknown messages, sends a help reply

### Requirements

| Credential | How to get |
|---|---|
| Telegram Bot Token | @BotFather → /newbot → copy token |
| Google Sheets OAuth2 | n8n Settings → Credentials → Google Sheets → Sign in |
| Slack Bot Token | api.slack.com/apps → create app → copy xoxb- token |

### Node-by-Node Explanation

#### Node 1: Telegram Trigger
```
Type: Telegram Trigger
Credential: your Telegram bot credential
Updates: Message
Why: Fires every time anyone sends your bot a message.
     Works in POLLING mode — no ngrok or public URL needed.
     n8n checks Telegram every 3 seconds.
Output fields used:
  {{ $json.message.chat.id }}        → where to reply
  {{ $json.message.from.first_name }} → sender's name
  {{ $json.message.text }}            → the full message text
```

#### Node 2: If — Is it a /ticket command?
```
Type: If
Condition:
  Value 1: {{ $json.message.text }}
  Operation: Starts With
  Value 2: /ticket
Why: We only process messages that start with /ticket.
     All other messages go to the false output (help reply).
True output (0):  process the ticket
False output (1): send help message
```

#### Node 3: Edit Fields — Build Ticket Object
```
Type: Edit Fields (Set)
Mode: Manual
Fields to set:
  ticketId:    TICK-{{ $now.toMillis() }}
               Why: millisecond timestamp = guaranteed unique ID
  user:        {{ $json.message.from.first_name }}
  chatId:      {{ $json.message.chat.id }}
               Why: save this now — needed for the reply later
  issue:       {{ $json.message.text.replace('/ticket ', '') }}
               Why: remove the command prefix, keep just the issue text
  status:      Open
  createdAt:   {{ $now.toFormat('dd/MM/yyyy HH:mm') }}
  priority:    Medium
               Why: default; can be changed in Sheets or by the agent
```

#### Node 4: Google Sheets — Log Ticket
```
Type: Google Sheets
Operation: Append Row
Spreadsheet: your IT tracker spreadsheet URL
Sheet: Tickets
Column mappings:
  Ticket ID  → {{ $json.ticketId }}
  User       → {{ $json.user }}
  Issue      → {{ $json.issue }}
  Status     → {{ $json.status }}
  Priority   → {{ $json.priority }}
  Created At → {{ $json.createdAt }}
  Resolved At → (leave empty)
Why: Creates a permanent audit trail. Can filter/sort in Sheets.
Setup: Create the spreadsheet first with these exact column headers in row 1.
```

#### Node 5: Slack — Notify #it-support
```
Type: Slack
Operation: Send Message
Channel: #it-support
Message:
  *New IT Ticket*
  Ticket ID: {{ $json.ticketId }}
  From: {{ $json.user }}
  Issue: {{ $json.issue }}
  Created: {{ $json.createdAt }}
Why: Alerts the IT team in real time.
Note: Bot must be /invite'd to #it-support channel first.
```

#### Node 6: Telegram — Reply to User
```
Type: Telegram
Operation: Send Message
Chat ID: {{ $json.chatId }}
Text (HTML):
  <b>Ticket Created</b>
  ID: <code>{{ $json.ticketId }}</code>
  Issue: {{ $json.issue }}
  Expected response: within 4 business hours.
Parse Mode: HTML
Why: Confirms to the user that their request was received.
```

#### Node 7: Telegram — Send Help (False branch)
```
Type: Telegram
Operation: Send Message
Chat ID: {{ $json.message.chat.id }}
Text:
  Send me your IT issue using:
  /ticket <describe your issue>
  Example: /ticket My VPN won't connect
Why: Guides users who send random messages.
```

### Testing
```
1. Start workflow (it activates polling mode)
2. Open Telegram → send to your bot: /ticket My laptop screen is flickering
3. Expected: ticket appears in Google Sheets, Slack notification, Telegram reply
4. Also test: send "hello" → should get help message
```

---

## Workflow 2 — Daily Morning Briefing

**Import file:** `workflows/json/02-daily-morning-briefing.json`

### What it does

Every weekday at 9 AM (IST), fetches open tasks from a Google Sheet and sends a structured briefing to Telegram and email.

### Requirements

| Credential | Source |
|---|---|
| Google Sheets OAuth2 | As above |
| Telegram Bot Token | As above |
| Gmail / SMTP | Settings → SMTP or Google OAuth2 |

### Node-by-Node Explanation

#### Node 1: Schedule Trigger
```
Type: Schedule Trigger
Trigger Interval: Custom (Cron)
Cron Expression: 0 9 * * 1-5
  0 = minute 0
  9 = hour 9 (9 AM)
  * = any day of month
  * = any month
  1-5 = Monday to Friday
Timezone (Add Option): Asia/Kolkata
Why timezone: Without this, "9" means 9 AM UTC = 2:30 PM IST.
               Always set timezone explicitly.
```

#### Node 2: Google Sheets — Read Open Tasks
```
Type: Google Sheets
Operation: Read Rows
Spreadsheet: your daily tasks spreadsheet
Sheet: Tasks
Filters:
  Column: Status
  Value: Open
Return All Matching: ON
Why: Only gets incomplete tasks. Completed ones are ignored.
```

#### Node 3: Code — Build Briefing Report
```javascript
// Type: Code | Mode: Run Once for All Items
const items = $input.all();
const today = new Date().toLocaleDateString('en-IN', {
  timeZone: 'Asia/Kolkata',
  weekday: 'long',
  day: 'numeric',
  month: 'long',
  year: 'numeric'
});

// Separate by priority
const high = items.filter(i => i.json.Priority === 'High');
const medium = items.filter(i => i.json.Priority === 'Medium');
const low = items.filter(i => i.json.Priority === 'Low');

const formatList = (arr) =>
  arr.length === 0
    ? '  None'
    : arr.map(i => `  - ${i.json.Task} (Due: ${i.json['Due Date'] || 'TBD'})`).join('\n');

const telegramText =
  `Good morning! Here is your briefing for ${today}\n\n` +
  `Total open tasks: ${items.length}\n\n` +
  `HIGH PRIORITY (${high.length})\n${formatList(high)}\n\n` +
  `MEDIUM PRIORITY (${medium.length})\n${formatList(medium)}\n\n` +
  `LOW PRIORITY (${low.length})\n${formatList(low)}`;

const htmlEmail =
  `<h2>Morning Briefing — ${today}</h2>` +
  `<p><strong>Total open tasks:</strong> ${items.length}</p>` +
  `<h3>High Priority (${high.length})</h3>` +
  (high.length ? `<ul>${high.map(i => `<li>${i.json.Task}</li>`).join('')}</ul>` : '<p>None</p>') +
  `<h3>Medium Priority (${medium.length})</h3>` +
  (medium.length ? `<ul>${medium.map(i => `<li>${i.json.Task}</li>`).join('')}</ul>` : '<p>None</p>');

return [{ json: { telegramText, htmlEmail, taskCount: items.length, date: today } }];
```

#### Node 4: Telegram — Send Briefing
```
Type: Telegram
Operation: Send Message
Chat ID: YOUR_PERSONAL_CHAT_ID
Text: {{ $json.telegramText }}
Parse Mode: None (plain text)
Why: Personal morning briefing directly in Telegram.
```

#### Node 5: Gmail — Send Email Briefing
```
Type: Gmail (or Send Email)
Operation: Send
To: shrinivas@example.com
Subject: Daily Briefing — {{ $json.date }}
Email Format: HTML
Message: {{ $json.htmlEmail }}
Why: Email version is more readable and archivable.
     Can be forwarded or referenced later.
```

---

## Workflow 3 — n8n Workflow Backup to GitHub

**Import file:** `workflows/json/03-n8n-backup-to-github.json`  
**Full guide:** [`workflows/01-save-workflows-to-github.md`](./01-save-workflows-to-github.md)

### What it does

Every night at 2 AM, exports all n8n workflows as JSON and commits them to a private GitHub repo.

### Requirements

| Credential | How to get |
|---|---|
| GitHub PAT | github.com → Settings → Fine-grained PAT → Contents: Read+Write |
| n8n API Key | n8n Settings → n8n API → Enable → Create key |
| Telegram Bot Token | For completion notification |

### Node Summary
```
Schedule (2 AM) → HTTP Request (n8n API, get all workflows)
→ Edit Fields (extract data[]) → Split Out (one item per workflow)
→ Loop Over Items
    → Code (build filename + base64 content)
    → GitHub: Get File (check if exists, On Error: Continue)
    → If (file.sha exists?)
        YES → GitHub: Edit File (with SHA)
        NO  → GitHub: Create File
→ Loop Done → Code (count results) → Telegram
```

**Key step — why SHA is needed for Edit:**
GitHub API requires the current file's SHA hash when updating. Without it, the API returns 409 Conflict. Get it from the Get File response: `{{ $json.sha }}`

---

## Workflow 4 — CareerForge Lead Qualifier

**Import file:** `workflows/json/04-careerforge-lead-qualifier.json`

### What it does

New lead submits a form (or webhook from website). The workflow:
1. Normalises input data
2. Scores the lead using rule-based logic (no AI needed)
3. Routes by tier: Hot / Warm / Cold
4. Hot leads: immediate Telegram alert + Gmail
5. Warm leads: add to email sequence
6. All leads: append to Google Sheets CRM

### Lead Scoring Logic (Pure Rules, No AI)

```javascript
// Code Node — scores lead without any AI
const lead = $json;
let score = 0;

// 1. Seniority (0–30 points)
const title = (lead.jobTitle || '').toLowerCase();
if (/director|vp|head|chief|cxo/.test(title)) score += 30;
else if (/manager|lead|principal/.test(title)) score += 20;
else if (/senior|architect|specialist/.test(title)) score += 15;
else score += 5;

// 2. Experience (0–25 points)
const years = parseInt(lead.yearsExperience) || 0;
if (years >= 10) score += 25;
else if (years >= 5) score += 15;
else if (years >= 2) score += 8;

// 3. Pain point match (0–20 points)
const pain = (lead.painPoint || lead.message || '').toLowerCase();
if (/job search|new role|salary|promotion|switch/.test(pain)) score += 20;
else if (/skill|upskill|certif|ai|cloud/.test(pain)) score += 12;
else score += 3;

// 4. Geography — target market (0–25 points)
const region = (lead.location || lead.country || '').toLowerCase();
if (/uae|dubai|abu dhabi|gcc|qatar|saudi|bahrain|kuwait|oman/.test(region)) score += 25;
else if (/india|mumbai|bangalore|chennai|hyderabad|pune|delhi/.test(region)) score += 15;
else score += 5;

const tier = score >= 80 ? 'Hot' : score >= 50 ? 'Warm' : 'Cold';
return [{ json: { ...lead, score, tier, qualifiedAt: new Date().toISOString() } }];
```

### Switch Node — Route by Tier
```
Mode: Rules
Routing field: {{ $json.tier }}
  Rule 1: equals Hot   → Output 0
  Rule 2: equals Warm  → Output 1
  Rule 3: equals Cold  → Output 2
```

### Hot Lead Actions
```
Telegram:
  HOT LEAD — Score: {{ $json.score }}/100
  Name: {{ $json.name }}
  Title: {{ $json.jobTitle }}
  Location: {{ $json.location }}
  Issue: {{ $json.painPoint }}
  Form: {{ $json.submittedAt }}

Gmail:
  Subject: CareerForge — New Hot Lead: {{ $json.name }}
  Body: personalised intro offering a free call
```

### Google Sheets — Log All Leads
```
Operation: Append or Update Row
Match column: email
Columns: Name | Email | Title | Company | Location | Score | Tier | Pain Point | Date
```

---

## Workflow 5 — Gmail to Application Tracker

**Import file:** `workflows/json/05-gmail-application-tracker.json`

### What it does

When a job application confirmation or recruiter reply arrives in Gmail, it is automatically logged to a Google Sheet tracker. Duplicate submissions (same email sender) update the existing row rather than creating a new one.

### Node-by-Node

#### Node 1: Gmail Trigger
```
Trigger On: New Email
Label/Folder: INBOX
Read Status: Unread only
Filters — Subject Contains: application OR interview OR recruiter OR shortlist
Why: Only process job-related emails, not everything.
     Adjust the subject filter to match what you actually receive.
```

#### Node 2: Edit Fields — Extract Data
```
senderEmail:   {{ $json.from.match(/<(.+)>/) ? $json.from.match(/<(.+)>/)[1] : $json.from }}
               Why: Extracts clean email from "Name <email>" format
senderName:    {{ $json.from.split('<')[0].trim().replace(/"/g, '') }}
subjectLine:   {{ $json.subject }}
receivedDate:  {{ $now.toFormat('dd/MM/yyyy') }}
snippet:       {{ $json.snippet }}
emailId:       {{ $json.id }}
```

#### Node 3: Code — Detect Stage
```javascript
// Classify email stage without AI
const subject = ($json.subjectLine || '').toLowerCase();
const body = ($json.snippet || '').toLowerCase();
const combined = subject + ' ' + body;

let stage = 'New Contact';
if (/interview|schedule|meet|call/.test(combined)) stage = 'Interview';
else if (/offer|selected|congratulations|pleased to/.test(combined)) stage = 'Offer';
else if (/regret|unfortunately|not moving|not selected/.test(combined)) stage = 'Rejected';
else if (/application received|thank you for applying/.test(combined)) stage = 'Applied';
else if (/recruiter|opportunities|open roles/.test(combined)) stage = 'Recruiter Outreach';

return [{ json: { ...$json, stage } }];
```

#### Node 4: Google Sheets — Append or Update
```
Operation: Append or Update Row
Match Column: Sender Email
Match Value: {{ $json.senderEmail }}
Columns:
  Sender Name  → {{ $json.senderName }}
  Sender Email → {{ $json.senderEmail }}
  Subject      → {{ $json.subjectLine }}
  Stage        → {{ $json.stage }}
  Last Updated → {{ $json.receivedDate }}
  Notes        → {{ $json.snippet }}
Why: If same sender emails again, it updates the row (tracks progression).
     New sender creates a new row.
```

#### Node 5: Gmail — Add Label
```
Operation: Add Label
Email ID: {{ $json.emailId }}
Label: n8n-logged
Why: Marks email as processed. Prevents re-processing on next trigger run.
     Create the label manually in Gmail first.
```

#### Node 6: Telegram — Notify
```
Text:
  Job Tracker Updated
  Stage: {{ $json.stage }}
  From: {{ $json.senderName }}
  Subject: {{ $json.subjectLine }}
Why: Real-time awareness of job search activity.
```

---

## Workflow 6 — ServiceNow Incident Monitor

> No JSON template — ServiceNow URL and auth varies per instance.
> Build this yourself using the steps below.

### What it does

Every 15 minutes, checks ServiceNow for newly opened P1/P2 incidents assigned to your team. Sends Telegram alert for new ones and logs to Google Sheets.

### Node Structure
```
Schedule Trigger (every 15 min)
    ↓
HTTP Request (ServiceNow REST API)
    ↓
Code (filter new incidents since last run)
    ↓
If (any new incidents?)
    ↓ YES
    Split Out (one item per incident)
    ↓
    Loop Over Items
      → Telegram (alert per incident)
      → Google Sheets (log)
    ↓ NO
    No Operation (stop gracefully)
```

### HTTP Request — ServiceNow API
```
Method: GET
URL:
  https://your-instance.service-now.com/api/now/table/incident
  ?sysparm_query=opened_at>javascript:gs.beginningOfLast15Minutes()
  ^assignment_group=YOUR_GROUP_SYS_ID
  ^priority<=2
  ^stateNOT IN6
  &sysparm_fields=number,short_description,priority,assigned_to,opened_at,state
  &sysparm_limit=50
Authentication: Basic Auth
  Username: your ServiceNow username
  Password: your ServiceNow password or API token
```

### Telegram Alert Format
```
SERVICENOW ALERT — {{ $now.toFormat('HH:mm dd MMM') }}

Incident: {{ $json.number }}
Priority: P{{ $json.priority }}
Description: {{ $json.short_description }}
Assigned to: {{ $json.assigned_to.display_value }}
Opened at: {{ $json.opened_at }}

View: https://your-instance.service-now.com/nav_to.do?uri=incident.do?sysparm_query=number={{ $json.number }}
```

---

## Workflow 7 — Recruiter Outreach Tracker (SPARK)

> Based on your SPARK methodology for LinkedIn recruiter outreach.

### What it does

Reads a Google Sheet of recruiters to contact. For each new contact (status = Pending), sends a personalised connection message draft to a Telegram message for review, then marks as Sent in Sheets.

### Node Structure
```
Manual Trigger (or Schedule, weekly Monday 9 AM)
    ↓
Google Sheets: Read Rows (Status = Pending)
    ↓
Filter: keep rows with outreach_count < 3 (max 3 touches)
    ↓
Loop Over Items (batch: 5)
    ↓
  Code: build SPARK message
  Telegram: Send and Wait (Approve / Edit / Skip)
    ↓
  If: Approved?
    YES → Google Sheets: Update Row (Status = Sent, Count+1, Date)
    NO  → Skip (No Operation)
```

### SPARK Message Builder Code
```javascript
const recruiter = $json;
// SPARK: Specific, Professional, Appreciative, Relevant, Kind
const message =
  `Hi ${recruiter.firstName}, ` +
  `I noticed your work recruiting for ${recruiter.specialisation} roles at ${recruiter.agency}. ` +
  `As a Senior IT Program Manager with ${recruiter.targetYears || '15'}+ years in ITAM/ITSM and ServiceNow across India and UAE, ` +
  `I would love to connect and explore any relevant opportunities.`;

// Enforce 300-char LinkedIn limit
const trimmed = message.length > 300 ? message.substring(0, 297) + '...' : message;
return [{ json: { ...recruiter, sparkMessage: trimmed, charCount: trimmed.length } }];
```

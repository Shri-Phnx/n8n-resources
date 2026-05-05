# Pure Automation Workflows — No AI Agent Required

> Author: Shrinivas Ramaprasad | May 2026
> All workflows use only Triggers, Core Nodes, and App Integrations.
> AI is optional where noted — always with a free/open-source fallback.
> Importable JSON files: `workflows/json/`

---

## Why Pure Automation?

- **Faster execution** — no LLM API call latency
- **No API costs** — completely free to run
- **More reliable** — no hallucination risk
- **Easier to debug** — deterministic output every time
- **Better for production** — rule-based logic is predictable

Use AI only when the task genuinely requires language understanding. Everything else: pure automation.

---

## Workflow 1 — IT Support Telegram Ticket Bot

**Trigger:** Telegram message starting with `/ticket`
**Use case:** Your actual ITSM workflow — users raise tickets via Telegram bot
**Import file:** `workflows/json/01-it-telegram-ticket-bot.json`

### Step-by-Step Node Build

```
Node 1: Telegram Trigger
  Updates: Message
  → Fires every time someone messages your bot

Node 2: If — Is it a /ticket command?
  Condition: {{ $json.message.text }} starts with /ticket
  [True] → continue
  [False] → Node 10 (send help message)

Node 3: Edit Fields — Build ticket object
  ticketId:   TICK-{{ $now.toMillis() }}
  userName:   {{ $json.message.from.first_name }}
  userId:     {{ $json.message.from.id }}
  chatId:     {{ $json.message.chat.id }}
  issue:      {{ $json.message.text.replace('/ticket ', '') }}
  status:     Open
  priority:   Medium
  createdAt:  {{ $now.toFormat('dd/MM/yyyy HH:mm') }}
  source:     Telegram

Node 4: Google Sheets — Append Row
  Spreadsheet: IT Support Tracker (your sheet URL)
  Sheet: Tickets
  Columns (map each):
    Ticket ID   → {{ $json.ticketId }}
    User Name   → {{ $json.userName }}
    Chat ID     → {{ $json.chatId }}
    Issue       → {{ $json.issue }}
    Status      → {{ $json.status }}
    Priority    → {{ $json.priority }}
    Created At  → {{ $json.createdAt }}
    Source      → {{ $json.source }}
  WHY: Creates a permanent audit trail viewable by anyone

Node 5: Slack — Send Message
  Channel: #it-support
  Text:
  🎫 *New IT Ticket*
  *ID:* {{ $json.ticketId }}
  *From:* {{ $json.userName }}
  *Issue:* {{ $json.issue }}
  *Time:* {{ $json.createdAt }}
  WHY: Notifies the IT team in real time

Node 6: Telegram — Send Message (confirmation to user)
  Chat ID: {{ $json.chatId }}
  Text:
  ✅ Your ticket has been created!

  🎫 *Ticket ID:* {{ $json.ticketId }}
  📝 *Issue:* {{ $json.issue }}
  ⏱ *Created:* {{ $json.createdAt }}

  We'll get back to you shortly.
  Parse Mode: Markdown

Node 10: Telegram — Send Help (false branch)
  Chat ID: {{ $json.message.chat.id }}
  Text: To create a ticket, use:
  /ticket describe your issue here
  Example: /ticket My laptop won't connect to VPN
```

### Google Sheets Setup

```
1. Create a new Google Sheet named: IT Support Tracker
2. Add header row (Row 1):
   A: Ticket ID | B: User Name | C: Chat ID | D: Issue
   E: Status    | F: Priority  | G: Created At | H: Source
3. Share sheet with your team
4. Copy the sheet URL and paste into Node 4
```

### How to Test

```
1. Start workflow → click Listen for test event
2. Open Telegram → message your bot: /ticket My screen is broken
3. n8n captures the data → check each node output
4. Verify: row added to Google Sheets, Slack message sent
5. Activate workflow when satisfied
```

---

## Workflow 2 — Daily ITAM / Status Report

**Trigger:** Schedule — every day at 9:00 AM IST
**Use case:** Morning briefing covering open tickets, ITAM alerts, pending actions
**Import file:** `workflows/json/02-daily-morning-briefing.json`

### Step-by-Step Node Build

```
Node 1: Schedule Trigger
  Trigger Interval: Every Day
  Hour: 9 | Minute: 0
  Timezone: Asia/Kolkata
  WHY: You start the day with full situational awareness

Node 2: Google Sheets — Read Rows (Open Tickets)
  Sheet: IT Support Tracker → Tickets tab
  Filter: Column = Status | Value = Open
  WHY: Get count and details of unresolved tickets

Node 3: Google Sheets — Read Rows (Pending Actions)
  Sheet: ITAM Tracker → Assets tab
  Filter: Column = Action Required | Value = Yes
  WHY: Surface any asset-level items needing attention

Node 4: Code Node (Run Once for All Items) — Compile Report
```

```javascript
// Node 4 Code — Compile Daily Report
const items = $input.all();

// Separate ticket and asset data by source field added in Edit Fields
// (Add Edit Fields after each Sheets node to tag source)
const tickets = items.filter(i => i.json._source === 'tickets');
const assets  = items.filter(i => i.json._source === 'assets');

const date = new Date().toLocaleDateString('en-IN', {
  timeZone: 'Asia/Kolkata',
  weekday: 'long', day: 'numeric', month: 'long', year: 'numeric'
});

const critical = tickets.filter(t => t.json.Priority === 'Critical').length;
const high     = tickets.filter(t => t.json.Priority === 'High').length;
const medium   = tickets.filter(t => t.json.Priority === 'Medium').length;
const assetAlerts = assets.length;

const report =
  `📊 *Daily IT Status — ${date}*\n\n` +
  `🎫 *Open Tickets: ${tickets.length}*\n` +
  `  🔴 Critical: ${critical}\n` +
  `  🟠 High: ${high}\n` +
  `  🟡 Medium: ${medium}\n\n` +
  `🖥️ *ITAM Alerts: ${assetAlerts}*\n\n` +
  (tickets.length === 0 ? '✅ All clear! No open tickets.' : '');

return [{ json: { report, ticketCount: tickets.length, assetAlerts } }];
```

```
Node 5: Telegram — Send Message
  Chat ID: your-personal-chat-id
  Text: {{ $json.report }}
  Parse Mode: Markdown

Node 6: Gmail — Send Email (optional)
  To: your@email.com
  Subject: Daily IT Status — {{ $now.toFormat('dd MMM yyyy') }}
  Format: HTML
  Message: <pre>{{ $json.report }}</pre>
```

---

## Workflow 3 — CareerForge Lead Qualifier

**Trigger:** Webhook (from CareerForge website form) or n8n Form
**Use case:** Score inbound leads automatically, route to right follow-up
**Import file:** `workflows/json/04-careerforge-lead-qualifier.json`

### Step-by-Step Node Build

```
Node 1: Webhook Trigger
  Path: careerforge-lead
  Method: POST
  Auth: Header Auth (X-CF-Secret)
  WHY: Form on your website POSTs here when someone signs up

Node 2: Edit Fields — Normalise data
  name:       {{ $json.body.name || $json.body.fullName }}
  email:      {{ $json.body.email.toLowerCase().trim() }}
  phone:      {{ $json.body.phone }}
  jobTitle:   {{ $json.body.jobTitle }}
  experience: {{ $json.body.yearsExperience }}
  location:   {{ $json.body.location }}
  painPoint:  {{ $json.body.painPoint || $json.body.message }}
  submittedAt: {{ $now.toISO() }}

Node 3: Code Node — Score the Lead (No AI)
```

```javascript
// Node 3 — Rule-based lead scoring
const lead = $json;
let score = 0;

// Job seniority (max 30 pts)
const title = (lead.jobTitle || '').toLowerCase();
if (/director|vp|head of|chief/.test(title))   score += 30;
else if (/manager|lead|architect/.test(title)) score += 20;
else if (/senior|specialist/.test(title))      score += 15;
else                                             score += 5;

// Experience (max 25 pts)
const yrs = parseInt(lead.experience) || 0;
if (yrs >= 10) score += 25;
else if (yrs >= 5) score += 15;
else if (yrs >= 2) score += 8;

// Location — UAE/GCC is target market (max 25 pts)
const loc = (lead.location || '').toLowerCase();
if (/uae|dubai|abu dhabi|doha|riyadh|gcc|qatar|bahrain|kuwait/.test(loc)) score += 25;
else if (/india|chennai|bangalore|mumbai/.test(loc)) score += 15;
else score += 5;

// Pain point relevance (max 20 pts)
const pain = (lead.painPoint || '').toLowerCase();
if (/job|salary|offer|career change/.test(pain)) score += 20;
else if (/skill|upskill|certif|learn/.test(pain)) score += 10;
else score += 5;

const tier = score >= 80 ? 'Hot' : score >= 50 ? 'Warm' : 'Cold';

return [{ json: { ...lead, score, tier } }];
```

```
Node 4: Switch — Route by Tier
  Routing Field: {{ $json.tier }}
  Rule 1: equals Hot  → Output 0
  Rule 2: equals Warm → Output 1
  Fallback (Cold)     → Output 2

[Hot branch — Output 0]
  Node 5a: HubSpot — Create/Update Contact
    Properties: firstname, lastname, email, jobtitle, phone
    CF properties: cf_score, cf_tier, cf_pain_point
  Node 6a: Telegram — Alert to you immediately
    Text: 🔥 HOT LEAD!
    Name: {{ $json.name }} | Score: {{ $json.score }}
    Title: {{ $json.jobTitle }} | Location: {{ $json.location }}
    Email: {{ $json.email }}
  Node 7a: Gmail — Send personalised intro email

[Warm branch — Output 1]
  Node 5b: HubSpot — Create/Update Contact
  Node 6b: HubSpot — Enrol in email sequence

[Cold branch — Output 2]
  Node 5c: HubSpot — Create Contact (low priority)
  → No immediate alert, nurture sequence only
```

---

## Workflow 4 — n8n Workflow Auto-Backup to GitHub

**Trigger:** Schedule — daily at 2:00 AM IST
**Use case:** Never lose a workflow — every workflow backed up to private GitHub repo
**Import file:** `workflows/json/03-n8n-backup-to-github.json`

Full step-by-step guide: `workflows/01-save-workflows-to-github.md`

```
Node 1: Schedule Trigger (2 AM daily)
Node 2: HTTP Request — GET all workflows (n8n API)
Node 3: Edit Fields — extract data array
Node 4: Split Out — one item per workflow
Node 5: Code — sanitise filename + base64 encode JSON
Node 6: GitHub — Get File (check if exists, On Error: Continue)
Node 7: Code — extract SHA or null
Node 8: If — file exists?
  [YES] → GitHub Edit (with SHA)
  [NO]  → GitHub Create
Node 9: Merge
Node 10: Code — compile summary
Node 11: Telegram — send backup report
```

---

## Workflow 5 — Gmail Application Tracker

**Trigger:** Gmail Trigger — new email in INBOX from recruiters
**Use case:** Track every job application automatically — no manual logging
**Import file:** `workflows/json/05-gmail-application-tracker.json`

### Step-by-Step Node Build

```
Node 1: Gmail Trigger
  Trigger On: New Email
  Label: INBOX
  Read Status: Unread only
  Poll Every: 1 Minute
  WHY: Catches recruiter replies, application confirmations automatically

Node 2: Filter — Skip internal emails
  Condition: {{ $json.from }} does not contain @yourcompany.com
  WHY: Avoid logging internal mail as job applications

Node 3: Code — Extract Application Data
```

```javascript
// Detect application-related emails
const subject = ($json.subject || '').toLowerCase();
const from = ($json.from || '').toLowerCase();
const body = ($json.text || $json.snippet || '').toLowerCase();

// Keywords that indicate job application activity
const appKeywords = [
  'application', 'applied', 'interview', 'position', 'role',
  'opportunity', 'cv', 'resume', 'shortlist', 'assessment'
];
const isJobRelated = appKeywords.some(k => subject.includes(k) || body.includes(k));

if (!isJobRelated) return [];  // Skip non-job emails

// Extract company name from sender domain
const emailMatch = from.match(/@([\w.-]+\.\w+)/);
const domain = emailMatch ? emailMatch[1] : 'Unknown';
const company = domain.split('.')[0].replace(/^(mail|noreply|careers|jobs)/, '').trim();

// Detect stage from keywords
let stage = 'Applied';
if (/interview|schedule|invite/.test(subject + body)) stage = 'Interview Scheduled';
else if (/shortlist|shortlisted/.test(subject + body)) stage = 'Shortlisted';
else if (/offer/.test(subject + body)) stage = 'Offer';
else if (/regret|unsuccessful|not moving|not proceed/.test(subject + body)) stage = 'Rejected';

return [{
  json: {
    emailId: $json.id,
    company: company.charAt(0).toUpperCase() + company.slice(1),
    subject: $json.subject,
    from: $json.from,
    stage,
    emailDate: $json.date,
    loggedAt: new Date().toLocaleDateString('en-IN', { timeZone: 'Asia/Kolkata' })
  }
}];
```

```
Node 4: Google Sheets — Append or Update Row
  Sheet: Job Applications
  Match Column: emailId
  WHY: Prevents duplicate rows if email is processed twice
  Column mappings:
    Email ID   → {{ $json.emailId }}
    Company    → {{ $json.company }}
    Subject    → {{ $json.subject }}
    From       → {{ $json.from }}
    Stage      → {{ $json.stage }}
    Date       → {{ $json.emailDate }}
    Logged At  → {{ $json.loggedAt }}

Node 5: Telegram — Notify (only for important stages)
  If: stage is Interview Scheduled OR Offer
  Text:
  📬 *Application Update*
  🏢 {{ $json.company }}
  📋 {{ $json.stage }}
  📧 {{ $json.subject }}
```

---

## Workflow 6 — ServiceNow Incident Monitor

**Trigger:** Schedule — every 15 minutes
**Use case:** Monitor ServiceNow for critical incidents and alert immediately
**Note:** Uses HTTP Request node — no native ServiceNow n8n node needed

### Step-by-Step Node Build

```
Node 1: Schedule Trigger
  Every: 15 Minutes
  WHY: Near-real-time monitoring without webhook dependency

Node 2: HTTP Request — Query ServiceNow Incidents
  Method: GET
  URL: https://your-instance.service-now.com/api/now/table/incident
  Authentication: Basic Auth (ServiceNow user)
  Query Parameters:
    sysparm_query: priority=1^state!=6^sys_created_onONLast 15 minutes
    sysparm_fields: number,short_description,priority,state,assigned_to,sys_created_on
    sysparm_limit: 20

Node 3: Filter — Only unassigned or P1/P2
  Condition: {{ $json.priority }} less than or equal to 2

Node 4: If — Any incidents found?
  Condition: {{ $input.all().length }} greater than 0
  [YES] → Node 5
  [NO]  → No Operation

Node 5: Code — Format Alert
```

```javascript
const incidents = $input.all();
const lines = incidents.map(i => {
  const pri = i.json.priority === '1' ? '🔴 P1' : '🟠 P2';
  const assigned = i.json.assigned_to?.display_value || 'Unassigned';
  return `${pri} [${i.json.number}] ${i.json.short_description}\nAssigned: ${assigned}`;
}).join('\n\n');

return [{
  json: {
    message: `⚠️ *${incidents.length} Critical Incident(s)*\n\n${lines}`,
    count: incidents.length
  }
}];
```

```
Node 6: Telegram — Send Alert
  Chat ID: your-itam-group-chat-id
  Text: {{ $json.message }}
  Parse Mode: Markdown

Node 7: Slack — Post to #incidents (optional parallel)
  Channel: #it-incidents
  Text: {{ $json.message }}
```

---

## Workflow 7 — Recruiter Outreach Tracker (SPARK)

**Trigger:** Manual (run when you want to do a batch outreach)
**Use case:** Track your LinkedIn recruiter outreach — who contacted, status, follow-ups

### Step-by-Step Node Build

```
Node 1: Manual Trigger

Node 2: Google Sheets — Read Rows (Recruiter List)
  Sheet: SPARK Outreach
  Filter: Status = Pending
  WHY: Only process recruiters not yet contacted

Node 3: Filter — Valid email present
  Condition: {{ $json.Email }} is not empty

Node 4: Code — Build SPARK personalised message
```

```javascript
// SPARK methodology: Situation, Problem, Action, Result, Knowledge
const recruiter = $json;

const message = [
  `Hi ${recruiter['First Name']},`,
  '',
  `I came across your profile while exploring ${recruiter.Agency} and noticed your focus on ${recruiter.Specialisation}.`,
  '',
  `I'm a Senior IT Program Manager with 15+ years in ITAM/ITSM, ServiceNow, and large-scale IT transformations. ` +
  `I hold PRINCE2 Practitioner, ITIL v4 MP, and multiple certifications, and I'm currently open to Senior/Head of IT roles in the UAE/GCC market (AED 24–28K range).`,
  '',
  `Would love to connect and explore if there are any relevant mandates. My profile: linkedin.com/in/shrinivas-r`,
  '',
  `Best regards,`,
  `Shrinivas Ramaprasad`
].join('\n');

return [{
  json: {
    ...recruiter,
    draftMessage: message,
    charCount: message.length
  }
}];
```

```
Node 5: Gmail — Send Outreach Email
  To: {{ $json.Email }}
  Subject: Senior IT Program Manager — Open to UAE Opportunities
  Message: {{ $json.draftMessage }}
  WHY: Personalised outreach at scale

Node 6: Google Sheets — Update Row
  Match: Email column
  Status: Contacted
  Contacted At: {{ $now.toFormat('dd/MM/yyyy') }}
  Draft Sent: {{ $json.draftMessage.substring(0, 100) }}...

Node 7: Wait — 2 seconds
  WHY: Avoid Gmail rate limiting between sends

Node 8: Telegram (after loop done)
  Text: ✅ Outreach batch complete
  Sent to {{ $items.length }} recruiters
```

---

## How to Import These Workflows

```
1. Download the JSON file from workflows/json/
2. In n8n: top-right menu (☰) → Import from File
3. Select the downloaded JSON file → Import
4. Review all nodes — update credentials
5. Test with manual trigger first
6. Activate when ready

OR paste JSON directly:
1. n8n → ☰ → Import from URL
2. Paste raw GitHub URL of the JSON file
```

# Google Products Integration — Complete Field Guide

> Author: Shrinivas Ramaprasad | May 2026
> Credential setup: `credentials/01-credentials-guide.md`

---

## Credential Setup (Once for All Google Nodes)

### Option A — Quick Sign-In (Personal)

```
1. n8n: Settings → Credentials → + Add → Gmail OAuth2 API
   (same credential works for Sheets, Drive, Calendar, Docs)
2. Click Sign in with Google → select account → Allow
3. Status: ✅ Connected
```

### Option B — Cloud Console (Workspace)

```
1. console.cloud.google.com → New Project: n8n-automation
2. Enable APIs: Gmail, Sheets, Drive, Calendar, Docs
3. OAuth consent screen: External, add test users
4. Credentials → OAuth client ID → Web app
   Redirect URI: https://your-n8n.com/rest/oauth2-credential/callback
5. Copy Client ID + Secret → paste in n8n credential → Sign in
```

---

## 1. Google Sheets — Every Field

**Spreadsheet ID:** from URL: `docs.google.com/spreadsheets/d/SPREADSHEET_ID/edit`

### Operation: Read Rows

| Field | Sample Value | What It Does | If Wrong |
|---|---|---|---|
| Spreadsheet | `https://docs.google.com/spreadsheets/d/1BxiM...` | Which spreadsheet | 404 not found |
| Sheet | `Sheet1` | Which tab (case-sensitive) | Sheet not found |
| Filters → Column | `Email` or `A` | Filter by this column | Returns all rows |
| Filters → Value | `active` | Match this value | Returns wrong rows |
| Range | `A1:F200` | Limit to range | Default: entire sheet |
| Return All | ON | Get all matches | OFF: first match only |

Output: `{ "Name": "Shrinivas", "Email": "s@x.com", "Status": "active" }`

Access: `{{ $json.Email }}` `{{ $json.Name }}`

### Operation: Append Row

| Field | Sample | What It Does |
|---|---|---|
| Spreadsheet | URL | Target spreadsheet |
| Sheet | `Leads` | Target tab |
| Column → Name | `Email` | Maps to header column |
| Column → Value | `{{ $json.email }}` | Data to write |
| Value Input Option | `USER_ENTERED` | Interprets formulas and date formats |

Sample column mappings:

```
Name:       {{ $json.name }}
Email:      {{ $json.email }}
Company:    {{ $json.company }}
Source:     Webhook
Created At: {{ $now.toFormat('dd/MM/yyyy HH:mm') }}
Status:     New
```

### Operation: Append or Update Row

| Field | Sample | What It Does |
|---|---|---|
| Match Column | `Email` | Find existing row by this column |
| Match Value | `{{ $json.email }}` | Value to look for |
| Columns | key-value pairs | Fields to set |

Behaviour: if row with matching email exists → update; else → append new row.

**✅ Best practices:**
- Always use header row 1 with clear column names
- Use `Append or Update` with email as match key for CRM-style data
- `USER_ENTERED` for dates and formulas

**⚠️ Common mistakes:**

| Mistake | Effect | Fix |
|---|---|---|
| Sheet name wrong case | Sheet not found | Names are case-sensitive: Sheet1 ≠ sheet1 |
| Column name mismatch | Data goes wrong column | Header names must EXACTLY match |
| Wrong spreadsheet URL | 404 error | Copy URL from browser address bar |

---

## 2. Gmail — Every Field

### Operation: Send Email

| Field | Type | Sample | Required | Notes |
|---|---|---|---|---|
| From Email | Email | `shrinivas@example.com` | Yes | Must match authorised account |
| From Name | Text | `Shrinivas Ramaprasad` | No | Display name in recipient's inbox |
| To | Email | `{{ $json.recipientEmail }}` | Yes | Comma-separate multiple |
| CC | Email | `manager@company.com` | No | Carbon copy |
| BCC | Email | `archive@company.com` | No | Blind carbon copy |
| Subject | Text | `Ticket #{{ $json.id }} Updated` | Yes | Empty subject looks like spam |
| Email Format | HTML / Text | `HTML` | Yes | HTML for rich formatting |
| Message | HTML | `<p>Dear {{ $json.name }},</p>...` | Yes | Supports full HTML |
| Attachments | Binary field | `data` | No | Binary from previous node |

Sample HTML email:

```html
<div style="font-family:Arial;max-width:600px">
  <h2>IT Ticket Updated</h2>
  <p>Dear {{ $json.name }},</p>
  <p>Ticket <b>#{{ $json.ticketId }}</b> status: <b>{{ $json.status }}</b></p>
  <p>Updated: {{ $now.toFormat('dd MMM yyyy HH:mm') }}</p>
  <p><a href="{{ $json.ticketUrl }}">View Ticket</a></p>
</div>
```

### Operation: Get Many (Read Emails)

| Field | Sample | What It Does |
|---|---|---|
| Return All | ON | Fetch all matching emails |
| Limit | `50` | Max to return |
| Filters → Read Status | `Unread` | Unread / Read / Any |
| Filters → Labels | `INBOX` | Filter by Gmail label |
| Filters → Sender | `noreply@stripe.com` | From specific sender |
| Filters → Subject | `Invoice` | Subject contains this |
| Filters → Received After | `{{ $now.minus({days:1}).toISO() }}` | Time filter |
| Download Attachments | ON | Include attachment binary |

Output: `{ id, subject, from, to, date, body.text, body.html, attachments[] }`

---

## 3. Google Drive — Every Field

### Operation: Upload File

| Field | Sample | What It Does |
|---|---|---|
| Name | `report-{{ $now.toFormat('yyyy-MM-dd') }}.pdf` | Filename in Drive |
| Parents | Folder ID from Drive URL | Which folder to upload to |
| Input Data Field | `data` | Binary field from previous node |
| Convert to Google Format | Toggle | .docx → Google Docs automatically |

**Get Folder ID:** Open folder in Drive → URL: `drive.google.com/drive/folders/FOLDER_ID`

### Operation: Download File

| Field | Sample | What It Does |
|---|---|---|
| File ID | `{{ $json.id }}` | Which file to download |
| Google File Conversion | PDF / XLSX / DOCX | Export format for Docs/Sheets |

---

## 4. Google Calendar — Every Field

### Operation: Create Event

| Field | Sample | Notes |
|---|---|---|
| Calendar | `primary` | Or custom: `xx@group.calendar.google.com` |
| Title | `Meeting: {{ $json.clientName }}` | Event title |
| Start | `{{ $json.date }}T09:00:00+05:30` | ISO with timezone offset |
| End | `{{ $json.date }}T10:00:00+05:30` | ISO with timezone offset |
| All Day | Toggle | Ignores time if enabled |
| Description | `Topic: {{ $json.topic }}` | HTML supported |
| Location | `Zoom: https://zoom.us/j/123` | Physical or virtual location |
| Attendees | `{{ $json.emails }}` | Array of email addresses |
| Send Updates | All | Sends calendar invites |
| Add Google Meet | Toggle | Auto-generates Meet link |

Output: `{ id, htmlLink, hangoutLink (Google Meet URL), status }`

**⚠️ Common Calendar mistakes:**

| Mistake | Effect | Fix |
|---|---|---|
| No timezone in ISO datetime | Event at wrong time | Always add offset: `T09:00:00+05:30` |
| Wrong calendar ID | Creates on wrong calendar | Check Calendar Settings → Integrate |

---

## 5. Google Docs — Every Field

### Operation: Create from Template

| Field | Sample | What It Does |
|---|---|---|
| Title | `Invoice #{{ $json.invNum }}` | New document title |
| Template Document URL | Existing Google Doc URL | Clone this doc as base |
| Folder ID | Drive folder ID | Where to save the new doc |

### Operation: Update (Replace Text)

| Field | Sample | What It Does |
|---|---|---|
| Document ID | `{{ $json.docId }}` | Which doc to update |
| Actions → Replace Text | Find: `{{CLIENT}}` Replace: `{{ $json.name }}` | Find and replace |

Invoice generation workflow:

```
Google Docs: Create from Template
    → Update: Replace {{CLIENT}} → {{ $json.clientName }}
    → Update: Replace {{AMOUNT}} → {{ $json.amount }}
    → Google Drive: Download as PDF
    → Gmail: Send with PDF attachment
```

---

## 6. Google Translate

| Field | Sample | Notes |
|---|---|---|
| Text | `{{ $json.userMessage }}` | Text to translate |
| Translate To | `en` | Target language code |
| Translate From | `auto` | Auto-detect if left blank |

Common language codes: `en`=English, `ta`=Tamil, `ar`=Arabic, `hi`=Hindi, `fr`=French, `de`=German

**Credential:** Google API Key (enable Cloud Translation API in Cloud Console)

---

## 7. Google Analytics

**Credential:** Google OAuth2 (Analytics scope)

| Field | Sample | Notes |
|---|---|---|
| Property ID | `123456789` | GA4 property ID from Analytics |
| Date Ranges | `2026-04-01` to `2026-04-30` | Report period |
| Dimensions | `pagePath` | What to group by |
| Metrics | `sessions`, `users` | What to measure |

---

## 8. YouTube

**Credential:** Google OAuth2 (YouTube scope)

### Operation: Upload Video

| Field | Sample | Notes |
|---|---|---|
| Title | `How to use n8n — Tutorial 1` | Video title |
| Description | `Complete guide to...` | Supports newlines |
| Tags | `n8n, automation` | Comma-separated |
| Privacy Status | `private` → then `public` | Always start private |
| Input Data Field | `data` | Binary video file |

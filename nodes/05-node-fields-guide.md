# n8n Node Fields — Complete Field-by-Field Reference

> **Author:** Shrinivas Ramaprasad  
> **Last updated:** April 2026  
> **Purpose:** Understand every field in every major n8n node — what it is, where to get the value, what it does, and what happens if you leave it empty.

---

## How to Read This Guide

Every node in n8n has two tabs:

| Tab | Purpose |
|-----|---------|
| **Parameters** | The actual configuration — credentials, resource, operation, and all the fields needed to make the node work |
| **Settings** | Universal options that apply to every node — retry, error handling, notes |

Within Parameters, fields are usually layered:
```
Credential  →  Resource  →  Operation  →  Dynamic fields for that operation
```

Changing Resource changes which Operations appear. Changing Operation changes which fields appear below it.

---

## Part 1 — Universal Settings Tab (Same for Every Node)

The Settings tab is identical across all n8n nodes. You only need to learn it once.

```
[ Parameters ]  [ Settings ← you are here ]

  Always Output Data          [toggle]
  Execute Once                [toggle]
  Retry On Fail               [toggle]
  On Error                    [ Stop Workflow ▼ ]
  Notes                       [ text area ]
  Display Note in Flow?       [toggle]
  Node version x.x (Latest)
```

### Field-by-Field

| Field | What It Is | What It Does | If Left Off / Default |
|-------|-----------|-------------|----------------------|
| **Always Output Data** | Toggle — force the node to always pass data downstream | When enabled: even if the node produces no results (e.g., empty search), it passes one empty item `{}` downstream so the workflow doesn't halt | **Default OFF** — if your node returns nothing, all downstream nodes stop executing. ⚠️ Turn ON when you want the workflow to continue even on empty results |
| **Execute Once** | Toggle — process only the first input item, ignore the rest | When enabled: if 50 items come in, the node runs only once (for item 0). Useful for notifications, one-time actions | **Default OFF** — node runs for every input item. If you accidentally leave this OFF on a "Send Email" node, it will send one email per input item |
| **Retry On Fail** | Toggle — automatically retry if the node fails | When enabled: shows Max Tries (default 3) and Wait Between Tries (default 1000ms) fields. Retries the API call on network errors or timeouts | **Default OFF** — a single failure stops the execution. ⚠️ Turn ON for HTTP requests, API calls, or anything that can have transient failures |
| **On Error** | Dropdown — what to do when this node throws an error | **Stop Workflow** — execution halts and shows an error. **Continue** — skip the failing item and continue with next. **Continue with Error Output** — routes the error to a special output branch for custom error handling | **Default: Stop Workflow** — any error kills the execution. Change to **Continue** if you want bulk operations to keep running despite individual failures |
| **Notes** | Text area — internal notes about what this node does | Visible only to workflow editors. Useful for documenting why a field is set a certain way, what the node does in context | **Optional** — leaving it empty has no functional impact. ✅ Best practice: always add notes to complex nodes |
| **Display Note in Flow** | Toggle — show the Notes text as a sticky annotation on the canvas | When enabled: a small note annotation appears next to the node on the canvas | **Default OFF** — notes are hidden. Turn ON for nodes that others might not understand at a glance |
| **Node version** | Display only — shows the installed version | Informational — tells you if you're on the latest version. If not latest, some features may be unavailable | N/A — cannot be changed here |

---

## Part 2 — Telegram Node (Complete Field Reference)

> Reference: Screenshots provided show the Telegram node v1.2 with `Send and Wait for Response` operation.

### How Telegram Authentication Works

Before filling any fields, you need a **Telegram Bot**. Here's how:

```
1. Open Telegram → search @BotFather
2. Send: /newbot
3. Enter a display name: e.g., My n8n Bot
4. Enter a username: e.g., my_n8n_bot (must end in 'bot')
5. BotFather sends you a TOKEN like: 7123456789:AAF_xyz...
6. Copy this token — you will paste it in n8n's Telegram credential
```

In n8n:
```
Settings → Credentials → Add credential → Search "Telegram" → paste token → Save
```

---

### Parameters Tab — Field by Field

#### Field: Credential to connect with

| Attribute | Detail |
|-----------|--------|
| **What it is** | Dropdown to select your saved Telegram Bot credential |
| **Where to get it** | Create a Telegram Bot via @BotFather → copy the API token → create credential in n8n Settings → Credentials |
| **What it does** | Authenticates all API calls this node makes to Telegram's API |
| **If left empty** | ❌ Node will not execute — error: "Credentials not found" |
| **Tip** | You can have multiple Telegram bots (credentials). Give each one a clear name like "Support Bot" or "Approval Bot" |

---

#### Field: Resource

| Option | What It Unlocks |
|--------|----------------|
| **Chat** | Manage chats — get chat info, leave chat, get chat members |
| **Callback** | Respond to inline button callback queries from users |
| **File** | Upload and manage files in Telegram chats |
| **Message** | Send, delete, edit, pin messages — the most commonly used resource |

| Attribute | Detail |
|-----------|--------|
| **What it is** | The Telegram API object type you want to work with |
| **Where to get it** | Choose based on your use case — most automations use **Message** |
| **What it does** | Determines which operations appear in the Operation field below |
| **If wrong value selected** | You'll see operations that don't match your intent — switch to Message for most use cases |

---

#### Field: Operation (Resource = Message)

| Operation | What It Does |
|-----------|-------------|
| **Delete Chat Message** | Delete a specific message by its message ID |
| **Edit Message Text** | Update the text of an already-sent message |
| **Pin Chat Message** | Pin a message in the chat |
| **Send Animation** | Send a GIF or MP4 animation file |
| **Send Audio** | Send an audio/music file |
| **Send Chat Action** | Show "typing...", "uploading photo..." status to users |
| **Send Document** | Send any file as a document attachment |
| **Send Location** | Send GPS coordinates |
| **Send Media Group** | Send multiple photos/videos as an album |
| **Send Message** | Send a plain text or HTML/Markdown message (fire-and-forget) |
| **Send Photo** | Send an image file |
| **Send Sticker** | Send a sticker |
| **Send and Wait for Response** | Send a message AND pause the workflow until the user replies |
| **Send Video** | Send a video file |
| **Send Voice** | Send a voice note |

| Attribute | Detail |
|-----------|--------|
| **What it is** | The specific action to perform on the chosen resource |
| **Where to get it** | Choose based on what your automation needs to do |
| **What it does** | Determines all the fields that appear below it |
| **If wrong operation** | Wrong fields will appear and API calls will fail |

---

#### Field: Chat ID (appears for most Message operations)

| Attribute | Detail |
|-----------|--------|
| **What it is** | The unique identifier of the Telegram chat, group, or channel you want to send to |
| **Where to get it** | **Option 1 — For bots in groups:** Add @RawDataBot or @userinfobot to the group → it posts the chat ID. **Option 2 — From workflow data:** When using a Telegram Trigger, the chat ID is in `{{ $json.message.chat.id }}`. **Option 3 — Your personal chat:** Send `/start` to your bot, then call `https://api.telegram.org/bot<TOKEN>/getUpdates` — your chat ID appears in the JSON. **Option 4 — Channels:** Forward any channel message to @userinfobot — it shows the channel ID (starts with -100) |
| **What it does** | Tells Telegram where to deliver the message |
| **If left empty** | ❌ Node fails — Telegram API error: "Bad Request: chat_id is empty" |
| **Common values** | Personal chat: positive number (e.g., `123456789`). Groups: negative number (e.g., `-987654321`). Channels: negative number starting with -100 |
| **Tip** | Use `{{ $json.message.chat.id }}` as a dynamic expression to reply to whoever triggered the workflow |

---

#### Field: Message (appears for Send Message, Send and Wait, etc.)

| Attribute | Detail |
|-----------|--------|
| **What it is** | The text content to send in the Telegram message |
| **Where to get it** | Type it directly (static) or use an expression `{{ $json.someField }}` to pull data from a previous node |
| **What it does** | This is the actual text the user will receive in Telegram |
| **If left empty** | ❌ For most operations, the node fails. For some, it sends an empty message which looks broken |
| **Formatting** | Supports **HTML** (`<b>bold</b>`, `<i>italic</i>`, `<code>code</code>`) or **Markdown** (`*bold*`, `_italic_`, `` `code` ``). Set parse mode in Options |
| **Expressions example** | `Hello {{ $json.firstName }}! Your ticket #{{ $json.ticketId }} has been updated.` |

---

#### Field: Response Type (appears for "Send and Wait for Response" only)

| Option | What It Means |
|--------|---------------|
| **Approval** | n8n adds Approve/Reject buttons inline in the message. User taps a button. Workflow resumes with their choice | 
| **Free Text** | n8n generates a unique form URL. User opens the URL and types a free-text response |
| **Custom Form** | You define a custom form with multiple fields. n8n generates a URL for the user to fill in |

| Attribute | Detail |
|-----------|--------|
| **What it is** | How the workflow waits for and collects the user's response |
| **Where to get it** | Choose based on what kind of input you need from the user |
| **What it does** | Pauses the entire workflow execution until the user responds (or the wait time expires) |
| **If not configured correctly** | The message sends but the workflow may not resume, or may resume with wrong data |

---

#### Field: Approval Options (appears when Response Type = Approval)

| Attribute | Detail |
|-----------|--------|
| **What it is** | The text labels for the Approve and Reject buttons |
| **Where to get it** | Type custom labels (e.g., "Yes, Proceed" and "No, Cancel") or leave as default |
| **What it does** | Customises the button labels the user sees in the Telegram message |
| **Default** | If no options added, n8n uses built-in Approve/Reject buttons |
| **Tip** | Add clear, action-oriented labels so users know exactly what they're approving |

---

#### Field: Options → Limit Wait Time

| Attribute | Detail |
|-----------|--------|
| **What it is** | Sets a maximum time the workflow will wait for a user response |
| **Where to get it** | Choose a duration (e.g., 1 Day, 2 Hours, 30 Minutes) |
| **What it does** | If the user doesn't respond within this time, the workflow resumes automatically and routes through the timeout output |
| **If not set** | The workflow waits indefinitely — could be paused forever if user never responds |
| **Tip** | Always set this for approval workflows. Handle the timeout output with a "Reminder" or "Escalation" branch |

---

#### Field: Options → Append n8n Attribution

| Attribute | Detail |
|-----------|--------|
| **What it is** | Toggle to add "Sent with n8n" text to the message |
| **What it does** | Appends a small attribution line to the message |
| **Default** | ON — shows attribution. Turn OFF for professional/production bots |

---

## Part 3 — Webhook Node — Field by Field

### Parameters Tab

#### Field: HTTP Method

| Attribute | Detail |
|-----------|--------|
| **What it is** | The HTTP method the webhook will accept |
| **Options** | GET, POST, PUT, PATCH, DELETE, HEAD |
| **Where to get it** | Check what method the calling system uses. Most integrations (Stripe, GitHub, Shopify) use **POST** |
| **If wrong method** | The webhook returns a 404 error to the caller — the workflow doesn't trigger |
| **Tip** | When unsure, use POST — it accepts a body payload which gives you more data |

#### Field: Path

| Attribute | Detail |
|-----------|--------|
| **What it is** | The URL path suffix that identifies this webhook. Forms part of the webhook URL |
| **Example** | Path `payment-received` → URL becomes `your-n8n.com/webhook/payment-received` |
| **Where to get it** | Type any meaningful name. Auto-generated UUID if left blank |
| **If left blank** | n8n auto-generates a random UUID path (hard to remember) |
| **Tip** | Use descriptive paths like `stripe-payment`, `github-push`, `form-submit` |

#### Field: Authentication

| Option | When to Use |
|--------|-------------|
| **None** | Only for internal/trusted sources. Never for public internet webhooks |
| **Basic Auth** | Username + password. Simple but less secure |
| **Header Auth** | A secret header value (e.g., `X-API-Key`). Most common for service webhooks |
| **JWT** | Cryptographically signed token. Most secure |

| Attribute | Detail |
|-----------|--------|
| **What it is** | How the webhook verifies that callers are authorised |
| **Where to get it** | Create a credential in n8n Settings → Credentials, then select it here |
| **If left as None** | Anyone who knows your webhook URL can trigger your workflow ⚠️ |

#### Field: Response Mode

| Option | Behaviour | Best For |
|--------|-----------|----------|
| **Immediately** | Returns 200 OK instantly, workflow runs in background | Stripe, GitHub — they don't wait for a response |
| **Last Node** | Waits for workflow to finish, responds with last node's output | Frontend apps that need a result |
| **Using Respond to Webhook Node** | You control exactly when and what to respond | Complex logic, custom status codes |

| Attribute | Detail |
|-----------|--------|
| **If wrong mode** | **Immediately** for a frontend app → client gets no data. **Last Node** for Stripe → Stripe waits too long and sends the event again |

---

## Part 4 — HTTP Request Node — Field by Field

### Parameters Tab

#### Field: Method
| What | GET (read), POST (create), PUT (full update), PATCH (partial update), DELETE (remove), HEAD (check if exists) |
|------|--------|
| **Where to get it** | Check the API documentation of the service you're calling |
| **If wrong** | API returns 405 Method Not Allowed |

#### Field: URL
| Attribute | Detail |
|-----------|--------|
| **What it is** | The full API endpoint URL |
| **Example** | `https://api.stripe.com/v1/customers` |
| **Expressions** | `https://api.example.com/users/{{ $json.userId }}` |
| **If empty** | ❌ Node fails immediately — URL is required |
| **Tip** | Never hardcode environment-specific URLs. Use n8n Variables for base URLs |

#### Field: Authentication
| Option | When to Use |
|--------|-------------|
| **None** | Public APIs without auth |
| **Predefined Credential Type** | n8n has a built-in credential for this service (recommended) |
| **Generic Credential Type** | Use Basic Auth, API Key, Bearer Token, OAuth2 manually |
| **If wrong** | API returns 401 Unauthorized |

#### Field: Send Body
| Attribute | Detail |
|-----------|--------|
| **What it is** | Toggle to include a request body (only relevant for POST/PUT/PATCH) |
| **Body Content Type** | JSON (most common), Form Data, Form URL-encoded, Binary, Raw |
| **If wrong content type** | API may return 400 Bad Request or ignore the body |

#### Field: Body Parameters
| Attribute | Detail |
|-----------|--------|
| **What it is** | Key-value pairs for the request body |
| **Where to get it** | API documentation — the required fields for the endpoint |
| **Expression example** | Key: `email`, Value: `{{ $json.email }}` |
| **If missing required fields** | API returns 400/422 with validation errors |

#### Field: Send Headers
| Attribute | Detail |
|-----------|--------|
| **What it is** | Custom HTTP headers to include |
| **Common headers** | `Content-Type: application/json`, `Authorization: Bearer token`, `X-API-Key: secret` |
| **If auth header missing** | API returns 401 Unauthorized |

#### Field: Send Query Parameters
| Attribute | Detail |
|-----------|--------|
| **What it is** | URL query string parameters (appended after `?` in the URL) |
| **Example** | `?page=1&limit=50` |
| **Where to get it** | API documentation — optional or required filter/pagination params |

#### Field: Options → Timeout
| Default | 300000 ms (5 minutes) |
|---------|-----|
| **When to change** | Long-running AI APIs — increase. Fast APIs — decrease to fail fast |
| **If exceeded** | Node throws ETIMEDOUT error |

#### Field: Options → Pagination
| Attribute | Detail |
|-----------|--------|
| **What it is** | Built-in pagination handler — automatically fetches all pages |
| **Types** | Offset-based, Cursor-based, Link header |
| **If not configured** | Only the first page of results is returned |

---

## Part 5 — Schedule Trigger — Field by Field

#### Field: Trigger Interval
| Option | When to Use |
|--------|-------------|
| **Every N Minutes** | Polling or frequent checks |
| **Every N Hours** | Hourly data syncs |
| **Every Day** | Daily reports, end-of-day processing |
| **Every Week** | Weekly summaries |
| **Every Month** | Monthly billing checks |
| **Custom (Cron)** | Complex schedules like "Every weekday at 9 AM" |

| Attribute | Detail |
|-----------|--------|
| **If wrong interval** | Workflow runs too often (wastes API quota) or too infrequently (data is stale) |

#### Field: Cron Expression (when Custom selected)
| Format | `second minute hour day month weekday` |
|--------|--------|
| **Example** | `0 9 * * 1-5` = 9 AM Monday–Friday |
| **Tools** | Use [crontab.guru](https://crontab.guru) to validate expressions |
| **If wrong** | Workflow may never fire, or fire at unexpected times |

#### Field: Timezone
| Attribute | Detail |
|-----------|--------|
| **What it is** | The timezone for interpreting the schedule |
| **Default** | n8n server timezone |
| **Where to get it** | Your timezone: `Asia/Kolkata` for IST, `Asia/Dubai` for UAE |
| **⚠️ Common mistake** | n8n server is in UTC — if you set "9 AM" without setting timezone, it fires at 9 AM UTC (2:30 PM IST). Always set timezone explicitly |

---

## Part 6 — Code Node — Field by Field

#### Field: Language
| Option | Detail |
|--------|--------|
| **JavaScript** | Node.js environment — access to `$json`, `$input`, `$node`, `DateTime`, `$workflow` |
| **Python** | Python 3 — access to `_input`, `_execution`, `datetime` |
| **Which to choose** | JavaScript is better supported with more n8n built-in helpers. Python is good if you're more comfortable with it |

#### Field: Mode
| Option | Behaviour |
|--------|----------|
| **Run Once for All Items** | The code block runs once. You must iterate manually using `$input.all()` |
| **Run Once for Each Item** | The code runs separately for each item. Use `$input.item` to access the current item |
| **If wrong mode** | **All Items with single-item code:** Only processes first item. **Each Item with iteration code:** Creates duplicate output |

#### Field: Code Editor
| Attribute | Detail |
|-----------|--------|
| **What it is** | The actual JavaScript/Python code to execute |
| **Required output format** | JavaScript: `return [{ json: { key: value } }]` Python: `return [{ 'json': { 'key': 'value' } }]` |
| **If wrong return format** | Node fails — "Data is not an Array" or "Wrong data type" error |
| **If no return statement** | Workflow stops — no items passed downstream |

**Common Code Node helpers (JavaScript):**
```javascript
// Access current item
$input.item.json.fieldName

// Access ALL input items
const items = $input.all();

// Access another node's output
$node['NodeName'].json.fieldName

// Current date (Luxon)
DateTime.now().toFormat('dd/MM/yyyy')

// Workflow metadata
$workflow.id
$execution.id
```

---

## Part 7 — If Node — Field by Field

#### Field: Conditions
| Attribute | Detail |
|-----------|--------|
| **What it is** | One or more conditions that determine which output branch items go to |
| **Structure** | Left side (value) + Operator + Right side (value) |
| **Data types** | String, Number, Boolean, Date/Time, Array, Object |

#### Field: Left Value
| Attribute | Detail |
|-----------|--------|
| **What it is** | The value to compare (usually a field from the incoming data) |
| **Expression** | `{{ $json.status }}` or `{{ $json.score }}` |
| **If wrong expression** | Condition evaluates against `undefined` — may cause unexpected routing |

#### Field: Operator
| Category | Operators |
|----------|----------|
| String | Equal, Not Equal, Contains, Does Not Contain, Starts With, Ends With, Regex, Is Empty, Is Not Empty |
| Number | Equal, Not Equal, Greater Than, Less Than, Greater or Equal, Less or Equal, Is Empty |
| Boolean | True, False, Is True, Is False |
| Date | Is Before, Is After, Is Same |
| Array | Contains, Does Not Contain, Is Empty, Length |

#### Field: Combine conditions
| Option | Meaning |
|--------|--------|
| **AND** | All conditions must be true |
| **OR** | Any one condition must be true |

#### Output:
- **Output 0 (true):** Items matching the condition go here
- **Output 1 (false):** Items NOT matching go here

| If condition misconfigured | Items might go to the wrong branch — always test with real data using "Execute step" |
|---------------------------|----|

---

## Part 8 — Edit Fields (Set) Node — Field by Field

#### Field: Mode
| Option | Behaviour |
|--------|----------|
| **Manual** | Visual UI — add/edit/remove fields one by one |
| **JSON** | Write the output as raw JSON — more powerful, more error-prone |

#### Field: Keep Only Set
| Attribute | Detail |
|-----------|--------|
| **What it is** | Toggle — when ON, ONLY the fields you define here are in the output. All other fields from the input are removed |
| **⚠️ Critical warning** | If you turn this ON and forget to include a field that downstream nodes need, those nodes will fail because the field no longer exists |
| **Best practice** | Leave OFF unless you specifically want to strip fields. When ON, always check what downstream nodes need |

#### Field: Fields to Set
| Attribute | Detail |
|-----------|--------|
| **Name** | The output field name (can be new or existing) |
| **Value** | Static text, number, boolean, or expression `{{ $json.existingField }}` |
| **Type** | String, Number, Boolean, Array, Object, Null |
| **If wrong type** | Downstream nodes expecting a number may fail if you pass a string version of the number |

---

## Part 9 — Google Sheets Node — Field by Field

#### Field: Credential
| Attribute | Detail |
|-----------|--------|
| **What it is** | Your Google account credential with Sheets access |
| **Where to get it** | n8n Settings → Credentials → Add → Google Sheets OAuth2. Follow the OAuth consent screen |
| **Scopes required** | `https://www.googleapis.com/auth/spreadsheets` |
| **If expired** | Node fails with 401. Re-authenticate the credential in Settings |

#### Field: Resource
| Option | When to Use |
|--------|-------------|
| **Document** | Create, copy, or delete entire spreadsheet files |
| **Sheet Within Document** | Read, write, append, clear data in a specific sheet tab |

#### Field: Operation (Sheet Within Document)
| Operation | What It Does | Required Fields |
|-----------|-------------|----------------|
| **Append or Update Row** | Adds a new row, or updates if a matching row exists | Sheet URL/ID, matching column, values |
| **Append Row** | Always adds a new row at the bottom | Sheet URL/ID, values |
| **Clear** | Deletes all data in a range | Sheet URL/ID, range |
| **Create** | Creates a new sheet tab | Sheet URL/ID, tab name |
| **Delete** | Deletes a sheet tab | Sheet URL/ID, tab name |
| **Read Rows** | Returns all rows from the sheet as n8n items | Sheet URL/ID, range |
| **Update Row** | Updates an existing row | Sheet URL/ID, row number or match column, values |

#### Field: Document ID/URL
| Attribute | Detail |
|-----------|--------|
| **What it is** | The Google Sheets document to use |
| **Where to get it** | Copy the URL from your browser: `https://docs.google.com/spreadsheets/d/SPREADSHEET_ID/edit` |
| **If wrong** | 404 error or data written to wrong spreadsheet |

#### Field: Sheet Name
| Attribute | Detail |
|-----------|--------|
| **What it is** | The specific tab/sheet within the document |
| **Default** | Usually `Sheet1` or first tab |
| **If wrong** | 400 error — sheet not found |

#### Field: Columns
| Attribute | Detail |
|-----------|--------|
| **What it is** | The column header names to map data to |
| **How it works** | n8n reads the first row as headers. Your data field names must match header names exactly (case-sensitive) |
| **If mismatch** | Data goes to wrong column or is ignored |

---

## Part 10 — Slack Node — Field by Field

#### Field: Credential
| How to create | Slack App → OAuth → Bot Token (starts with `xoxb-`) → paste in n8n credential |
|---------------|----|
| **Permissions needed** | `chat:write`, `channels:read`, `users:read` depending on operations |
| **If wrong scopes** | Slack API returns `missing_scope` error |

#### Field: Resource → Message → Operation: Send

#### Field: Channel
| Attribute | Detail |
|-----------|--------|
| **What it is** | The Slack channel to send to |
| **Format** | `#channel-name` or Channel ID (e.g., `C01234567`) |
| **Where to get Channel ID** | Right-click channel → View Channel Details → scroll to bottom |
| **If bot not in channel** | Error: `not_in_channel`. Invite the bot to the channel first: `/invite @YourBotName` |

#### Field: Message Text
| Attribute | Detail |
|-----------|--------|
| **What it is** | The message content |
| **Supports** | Slack mrkdwn formatting: `*bold*`, `_italic_`, `` `code` ``, `>quote`, `<URL\|link text>` |
| **Expressions** | `{{ $json.message }}` or `Ticket #{{ $json.ticketId }} created by {{ $json.user }}` |
| **If empty** | Slack API error: `no_text` |

#### Field: Attachments / Blocks
| Attribute | Detail |
|-----------|--------|
| **What it is** | Structured message components — buttons, images, sections |
| **Where to get it** | Build blocks at [Slack Block Kit Builder](https://app.slack.com/block-kit-builder) |
| **Tip** | Use blocks for interactive messages with buttons and approval workflows |

---

## Part 11 — Send Email (SMTP) Node — Field by Field

#### Field: Credential
| Provider | SMTP Settings |
|----------|---------------|
| **Gmail** | Host: `smtp.gmail.com`, Port: 587, User: your@gmail.com, Pass: App Password (not your Google password) |
| **Outlook** | Host: `smtp.office365.com`, Port: 587 |
| **Custom** | Get from your email provider's settings page |

| **If wrong settings** | Connection refused or authentication failed |

#### Field: From Email
| Must match | The SMTP account's authenticated email address |
|------------|----|
| **If mismatched** | Some providers reject with `553 Sender verify failed` |

#### Field: To Email
| Attribute | Detail |
|-----------|--------|
| **Multiple recipients** | Separate with commas: `a@example.com, b@example.com` |
| **Expression** | `{{ $json.recipientEmail }}` |
| **If empty** | ❌ Node fails — recipient is required |

#### Field: Subject
| Tip | Always include — empty subject looks like spam |
|-----|----|

#### Field: Body (Email Content)
| Type | When to Use |
|------|-------------|
| **Text** | Plain text — no formatting |
| **HTML** | Rich formatting, images, buttons |
| **If wrong type** | HTML tags appear as raw text if Plain Text mode is used |

---

## Part 12 — Wait Node — Field by Field

#### Field: Resume (the main mode selector)
| Option | Behaviour |
|--------|----------|
| **Time Interval** | Pause for N seconds/minutes/hours/days, then resume automatically |
| **At Specific Time** | Pause until a specific date and time |
| **Webhook** | Pause until an external HTTP call is made to a generated URL |

#### Field: Amount + Unit (Time Interval mode)
| Attribute | Detail |
|-----------|--------|
| **What it is** | How long to pause |
| **Example** | Amount: `30`, Unit: `Minutes` — pauses for 30 minutes |
| **If too long** | Execution sits in "waiting" state — check via Executions tab |
| **Max wait** | No hard limit, but very long waits may be affected by server restarts |

#### Field: Webhook (Webhook Resume mode)
| Attribute | Detail |
|-----------|--------|
| **What it is** | n8n generates a unique resume URL for this specific execution |
| **Where to find it** | Available as `{{ $execution.resumeUrl }}` in the workflow after the Wait node |
| **How to use** | Send this URL to a user in an email or Telegram message. When they click a link to this URL, the workflow resumes |
| **If URL not called** | Workflow stays paused forever (unless Limit Wait Time is set) |

---

## Part 13 — Merge Node — Field by Field

#### Field: Mode
| Mode | What It Does | When to Use |
|------|-------------|-------------|
| **Append** | Combines all items from both inputs into one stream | Combining results from parallel branches |
| **Combine** | Merges items from two inputs pairwise or by matching key | Enriching data (add column B to column A) |
| **SQL Query** | Join-like SQL operation on two data sets | Complex multi-table style merges |
| **Choose Branch** | Select which branch's output to use | Conditional output |
| **Wait** | Holds items from one branch until both branches have data | When two async branches need to sync |

#### Field: How to Merge Data (when Mode = Combine)
| Option | Behaviour |
|--------|----------|
| **By Position** | First item from Input 1 merges with first item from Input 2, etc. |
| **By Matching Fields** | Match items where a specific field value is the same (like a JOIN on a key) |
| **All Possible Combinations** | Every item from Input 1 paired with every item from Input 2 |

| **⚠️ Input order matters** | Input 1 and Input 2 are determined by which connector you use — the top connector is always Input 1 |

---

## Part 14 — AI Agent Node — Field by Field

#### Field: System Prompt
| Attribute | Detail |
|-----------|--------|
| **What it is** | Instructions that define the agent's role, personality, and behaviour |
| **Where to get it** | Write it yourself — describe the role, what the agent can/cannot do, output format expectations |
| **If empty** | Agent has no context — may give generic or off-topic responses |
| **Tip** | Always include today's date via `{{ $now.toFormat('dd MMMM yyyy') }}` so the agent reasons correctly about time |

#### Field: Prompt (User Message)
| Attribute | Detail |
|-----------|--------|
| **What it is** | The actual user question or task for the agent |
| **Default** | `{{ $json.chatInput }}` — pulls from Chat Trigger |
| **If static text** | Every run sends the same prompt — only useful for scheduled summarisation tasks |

#### Field: Agent Type
| Type | When to Use |
|------|-------------|
| **Tools Agent** | ✅ Default — best for most use cases. Agent decides which tools to call |
| **Conversational** | Simple back-and-forth chat without tool calling |
| **ReAct** | Older reasoning format — use Tools Agent instead |
| **OpenAI Functions** | Specifically uses OpenAI's function calling format |
| **SQL** | For natural language → SQL query generation |

#### Field: Require Specific Output Format
| Attribute | Detail |
|-----------|--------|
| **What it is** | Force the agent to output a specific JSON schema |
| **When to use** | When downstream nodes need structured data (not free text) |
| **If enabled with wrong schema** | LLM may hallucinate fields or fail to comply |

#### Field: Chat Model (sub-node)
| Attribute | Detail |
|-----------|--------|
| **What it is** | Which LLM powers the agent |
| **Connection** | Click the Chat Model port on the AI Agent node → add model sub-node |
| **If not connected** | ❌ Workflow fails — AI Agent cannot run without an LLM |

#### Field: Memory (sub-node — optional)
| Attribute | Detail |
|-----------|--------|
| **What it is** | Stores conversation history so the agent remembers previous messages |
| **If not connected** | Agent has no memory — treats every message as a fresh conversation |
| **Session ID** | Use `{{ $sessionId }}` to group messages by conversation |

#### Field: Tools (sub-nodes — optional)
| Attribute | Detail |
|-----------|--------|
| **What it is** | Capabilities the agent can use (HTTP calls, search, calculations, etc.) |
| **If no tools** | Agent can only respond with text — cannot take actions |
| **Best practice** | Give each tool a clear description so the agent knows when to use it |

---

## Part 15 — Loop Over Items (Split in Batches) — Field by Field

#### Field: Batch Size
| Attribute | Detail |
|-----------|--------|
| **What it is** | How many items to process per batch iteration |
| **Default** | 1 — process one item at a time |
| **When to increase** | When the API supports batch requests and processing individually is slow |
| **When to keep at 1** | Rate-limited APIs — process one, wait, process next |
| **⚠️ Common mistake** | Setting batch size too high hits API rate limits — all items in the batch fail |

#### Outputs:
| Output | Behaviour |
|--------|----------|
| **Output 0 (loop body)** | Current batch items — connect your processing nodes here |
| **Output 1 (done)** | Fires only when all items have been processed — connect post-loop nodes here |

| **If Output 1 not connected** | Workflow technically completes but you have no post-loop action |

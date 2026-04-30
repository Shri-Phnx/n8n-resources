# n8n Core & Trigger Nodes — Detailed Reference

> **Author:** Shrinivas Ramaprasad  
> **Last updated:** April 2026

---

## Part 1 — Trigger Nodes

Trigger nodes start a workflow. Every workflow must begin with exactly one trigger. They self-activate based on an event, schedule, or manual action.

---

### Manual Trigger

| Attribute | Detail |
|-----------|--------|
| **Purpose** | Run the workflow manually from the n8n UI |
| **When to use** | Testing, one-off data migrations, demo runs |
| **How to use** | Add to canvas → click **"Execute workflow"** button |
| **Output** | One empty item `{}` — downstream nodes receive this as input |
| **Key setting** | None — it is a zero-config trigger |

---

### Schedule Trigger

| Attribute | Detail |
|-----------|--------|
| **Purpose** | Run workflow on a recurring schedule |
| **Supports** | Every N minutes/hours, specific days/times, cron expressions |
| **Cron syntax** | `0 9 * * 1-5` = Every weekday at 9 AM |

**Common cron patterns:**

| Pattern | Meaning |
|---------|---------|
| `0 * * * *` | Every hour |
| `0 9 * * *` | Every day at 9 AM |
| `0 9 * * 1-5` | Weekdays at 9 AM |
| `0 0 1 * *` | First day of every month |
| `*/15 * * * *` | Every 15 minutes |

**Key settings:**
- **Trigger Interval** — Simple picker (every N minutes/hours/days/weeks)
- **Cron Expression** — Advanced custom schedule
- **Timezone** — Always set explicitly; default may not match your region

---

### Email Trigger (IMAP)

| Attribute | Detail |
|-----------|--------|
| **Purpose** | Fire when a new email arrives in an IMAP mailbox |
| **Credential** | IMAP credentials (server, port, user, password/OAuth) |
| **Filters** | Mailbox folder, read/unread, subject contains, from address |
| **Output** | Email fields: `from`, `to`, `subject`, `text`, `html`, `date`, attachments |
| **Poll interval** | Configurable — how often n8n checks for new mail |

**Supports:** Gmail (via App Password or OAuth), Outlook, Yahoo, any IMAP server

---

### n8n Form Trigger

| Attribute | Detail |
|-----------|--------|
| **Purpose** | Create a hosted web form — submission triggers the workflow |
| **Form URL** | n8n generates a unique URL you share with users |
| **Field types** | Text, number, email, date, dropdown, file upload, checkboxes |
| **Output** | Each form field maps to a JSON property |
| **Use cases** | Lead capture, internal requests, approval forms, onboarding |

---

### Chat Trigger

| Attribute | Detail |
|-----------|--------|
| **Purpose** | Exposes a chat interface that triggers an AI agent workflow |
| **Interface** | n8n-hosted chat window or embeddable widget |
| **Output** | `chatInput` (user message), `sessionId` (conversation ID) |
| **Use with** | AI Agent node for conversational workflows |
| **Authentication** | None (public), or Basic Auth |

---

### Error Trigger

| Attribute | Detail |
|-----------|--------|
| **Purpose** | Global error handler — fires when any other workflow fails |
| **How to use** | Set this as the error workflow in **Settings → Error workflow** |
| **Output** | `workflow`, `execution`, `error` — full error context |
| **Common actions** | Send Slack/email alert, log to Airtable, create Jira ticket |

---

### Activation Trigger

| Attribute | Detail |
|-----------|--------|
| **Purpose** | Fire when a workflow is activated or deactivated |
| **Use case** | Bootstrap data on activation, cleanup on deactivation |
| **Events** | `activate`, `deactivate` |

---

## Part 2 — Core Nodes (Detailed)

---

### HTTP Request

The most powerful and versatile node in n8n — calls any REST, SOAP, or GraphQL API.

| Parameter | Options / Detail |
|-----------|------------------|
| **Method** | GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS |
| **URL** | Static or dynamic with expressions `{{ $json.url }}` |
| **Authentication** | None, Basic Auth, Header Auth, OAuth1/2, Digest, API Key, JWT |
| **Body** | JSON, Form-data, Form URL-encoded, Binary, Raw/Custom |
| **Headers** | Any custom headers |
| **Query params** | Key-value pairs or raw query string |
| **Response** | JSON (auto-parse), Text, File (binary) |
| **Options** | Follow redirects, timeout, proxy, SSL ignore, batching |
| **Pagination** | Built-in pagination — offset/cursor/page-based |

**When to use over dedicated nodes:** When n8n doesn't have a specific node for a service, or when a service node doesn't expose a specific API endpoint.

**Example — calling NVIDIA NIM API:**
```json
Method: POST
URL: https://integrate.api.nvidia.com/v1/chat/completions
Headers: Authorization: Bearer {{ $credentials.nvidiaNimApiKey }}
Body (JSON):
{
  "model": "meta/llama-3.1-70b-instruct",
  "messages": [{"role": "user", "content": "{{ $json.userMessage }}"}],
  "max_tokens": 1024
}
```

---

### Code Node

Run custom JavaScript or Python directly inside a workflow.

| Parameter | Detail |
|-----------|--------|
| **Language** | JavaScript (Node.js) or Python |
| **Mode** | Run Once for All Items / Run Once for Each Item |
| **Input** | Access via `$input.all()`, `$input.first()`, `$input.item` |
| **Output** | Must return an array of objects: `return [{ json: { ... } }]` |

**JavaScript example — transform and filter:**
```javascript
const items = $input.all();
return items
  .filter(item => item.json.status === 'active')
  .map(item => ({
    json: {
      id: item.json.id,
      fullName: `${item.json.firstName} ${item.json.lastName}`,
      email: item.json.email.toLowerCase()
    }
  }));
```

**Python example:**
```python
output = []
for item in _input.all():
    data = item.json
    output.append({
        'json': {
            'name': data['name'].upper(),
            'score': data['score'] * 1.1
        }
    })
return output
```

**Built-in helpers in JS:**
- `$json` — current item's JSON
- `$input` — input data methods
- `$node['NodeName'].json` — access other node's output
- `DateTime` — Luxon library (date operations)
- `$workflow` — workflow metadata
- `$execution` — execution metadata

---

### If Node

Splits workflow into two branches based on a condition.

| Parameter | Detail |
|-----------|--------|
| **Conditions** | One or more conditions combined with AND/OR |
| **Operators** | Equal, Not Equal, Contains, Regex, Greater/Less than, Is Empty, etc. |
| **Data types** | String, Number, Boolean, Date, Array, Object |
| **Output** | Two outputs — `true` (items matching) and `false` (non-matching) |

**Example condition:** `{{ $json.score }}` `greater than` `80`

---

### Switch Node

Routes items to one of many output paths based on a value.

| Parameter | Detail |
|-----------|--------|
| **Mode** | Rules (match conditions) or Expression (return output index) |
| **Rules** | Each rule has a condition and maps to a numbered output |
| **Default** | Items not matching any rule go to the fallback output |
| **Use case** | Route by status, region, type, priority, category |

---

### Merge Node

Combines data from two or more incoming branches.

| Mode | Behaviour |
|------|-----------|
| **Append** | Concatenate all items from both inputs |
| **Combine** | Merge items from both inputs into one (by position or key) |
| **SQL Query** | Run a SQL-like query to join items |
| **Choose Branch** | Output items from one specific branch |
| **Wait** | Wait for both branches to complete before merging |

---

### Edit Fields (Set) Node

Add, update, or remove fields on each item.

| Feature | Detail |
|---------|--------|
| **Add field** | New field name + value (static or expression) |
| **Update field** | Override existing field value |
| **Remove field** | Delete a field from the item |
| **Include other fields** | Keep all existing fields (or discard them) |
| **Mode** | Manual (UI fields) or JSON (write raw JSON) |

---

### Filter Node

Keep only items matching a condition; discard the rest.

| Feature | Detail |
|---------|--------|
| **Conditions** | Same condition builder as the If node |
| **Combine with** | AND / OR logic |
| **Output** | Single output — only matching items pass through |

---

### Loop Over Items (Split in Batches)

Process items in chunks; ideal for rate-limited APIs.

| Parameter | Detail |
|-----------|--------|
| **Batch size** | Number of items per batch |
| **Output 0** | Items in the current batch (feeds into loop body) |
| **Output 1** | Signals loop completion (connect to post-loop nodes) |

---

### Wait Node

Pause workflow execution temporarily.

| Mode | Detail |
|------|--------|
| **Time Interval** | Pause for N seconds/minutes/hours/days |
| **Specific Time** | Resume at a set date/time |
| **Webhook** | Pause until an external HTTP request resumes it |

**Webhook resume URL** is generated automatically — share it with external systems to resume the workflow on demand.

---

### Send Email Node

| Parameter | Detail |
|-----------|--------|
| **Credential** | SMTP (Gmail, Outlook, any SMTP server) |
| **To / CC / BCC** | Static or dynamic from data |
| **Subject** | Static or expression |
| **Body** | Plain text or HTML |
| **Attachments** | Binary data from previous nodes (e.g., from Convert to File) |

---

### Respond to Webhook Node

Send a custom HTTP response back to a webhook caller — must be used when **Response Mode** is set to `Using Respond to Webhook Node`.

| Parameter | Detail |
|-----------|--------|
| **Respond with** | JSON, Text, Binary, No Data, Redirect |
| **Status code** | Any HTTP status (200, 201, 400, etc.) |
| **Headers** | Custom response headers |
| **Data** | Response body — static or dynamic from workflow data |

---

### Date & Time Node

| Operation | Detail |
|-----------|--------|
| **Format** | Convert date to any string format |
| **Parse** | Parse a date string into a DateTime object |
| **Add / Subtract** | Date arithmetic |
| **Get current time** | Current timestamp in any timezone |
| **Round** | Round to nearest minute/hour/day |

---

### Execute Sub-workflow Node

Call another n8n workflow from within a workflow.

| Parameter | Detail |
|-----------|--------|
| **Workflow ID** | Select the target workflow |
| **Source of data** | Input items or fixed parameters |
| **Wait for result** | Synchronous (wait for response) or async (fire and forget) |
| **Credentials** | Use current workflow's credentials |

**Use case:** Modular design — shared utilities like "Send alert notification" or "Update CRM record" that many workflows call.

---

### Stop And Error Node

| Parameter | Detail |
|-----------|--------|
| **Error message** | Custom message to surface in execution log |
| **Error object** | Optional structured error data |
| **Use case** | Validation gates — stop if required data is missing |

---

## Part 3 — Important Node Combinations

| Pattern | Nodes Used | Use Case |
|---------|------------|----------|
| **Conditional branch** | If → (nodes A) + (nodes B) → Merge | Route by status or category |
| **Batch processing** | Loop Over Items → HTTP Request → Loop | Avoid API rate limits |
| **Error handling** | Error Trigger → Slack → Stop | Global alert on any failure |
| **Approval gate** | Wait (webhook) → If → Continue/Stop | Human-in-the-loop approval |
| **Scheduled ETL** | Schedule → HTTP Request → Filter → Google Sheets | Regular data sync |
| **Form-to-CRM** | Form Trigger → HubSpot → Gmail | New lead capture |
| **Sub-workflow fan-out** | Split Out → Execute Sub-workflow | Parallel processing |

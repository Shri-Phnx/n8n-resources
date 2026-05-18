# n8n Core Action Nodes — Detailed Reference

> **Author:** Shrinivas Ramaprasad
> **Updated:** May 2026
> **Source:** n8n Docs + N8N_All_Nodes.xlsx (67 Core Nodes verified)

> ⚠️ = Known issue, common pitfall, or important warning to be aware of before using this node.

---

## HTTP Request ⚠️

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

**⚠️ Common pitfalls:**
- Forgetting pagination on large datasets — enable **Pagination** under Options
- SSL certificate errors — toggle "Allow Unauthorized Certs" in Options for testing only (not production)
- Rate limits — use Loop Over Items with a Wait node between batches
- Dynamic URLs must be in expressions: `{{ 'https://api.example.com/users/' + $json.userId }}`

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

## Code Node

Run custom JavaScript or Python directly inside a workflow.

| Parameter | Detail |
|-----------|--------|
| **Language** | JavaScript (Node.js) or Python |
| **Mode** | Run Once for All Items / Run Once for Each Item |
| **Input** | Access via `$input.all()`, `$input.first()`, `$input.item` |
| **Output** | Must return an array of objects: `return [{ json: { ... } }]` |
| **⚠️ Warning** | Remove Debug Helper nodes before production — they add mock data |

**Code Node Modes:**

| Mode | When to Use | Access Data |
|------|-------------|-------------|
| Run Once for All Items | Aggregate, group, build CSV | `$input.all()` |
| Run Once for Each Item | Transform each item | `$json` |

**Required return format:**
```javascript
// Always return array of objects with 'json' key
return [{ json: { field: 'value', count: 42 } }];

// Multiple items:
return items.map(i => ({ json: { ...i.json, processed: true } }));
```

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

**Built-in JavaScript variables:**

| Variable | Returns | Sample |
|----------|---------|--------|
| `$json` | Current item data | `{ name: 'Shrinivas', score: 95 }` |
| `$input.all()` | All items as array | `[{ json: {...} }, ...]` |
| `$input.first()` | First item | `{ json: {...} }` |
| `$node['Name'].json` | Another node's output | `{ id: 123 }` |
| `$now` | Current time (Luxon) | Luxon DateTime object |
| `DateTime` | Luxon library | Date operations |
| `$vars.MY_VAR` | n8n Variable value | `'https://api.example.com'` |
| `$workflow.name` | Workflow name | `'Daily Report'` |
| `$execution.id` | Execution ID | `'ex_xyz789'` |

**⚠️ Common Code Node mistakes:**

| Mistake | Error | Fix |
|---------|-------|-----|
| `return { name: 'x' }` | "Data is not an array" | `return [{ json: { name: 'x' } }]` |
| `return []` | Empty — downstream gets nothing | Return `[{ json: {} }]` if needed |
| `$json.user.name` when user is null | TypeError | `$json.user?.name ?? 'Unknown'` |

---

## Edit Fields (Set) ⚠️

Add, update, or remove fields on each item.

| Feature | Detail |
|---------|--------|
| **Add field** | New field name + value (static or expression) |
| **Update field** | Override existing field value |
| **Remove field** | Delete a field from the item |
| **Include other fields** | Keep all existing fields (or discard them) |
| **Mode** | Manual (UI fields) or JSON (write raw JSON) |
| **⚠️ WARNING** | "Keep Only Set" mode deletes ALL fields not explicitly listed — always verify this setting |

---

## Filter Node

Keep only items matching a condition; discard the rest.

| Feature | Detail |
|---------|--------|
| **Conditions** | Same condition builder as the If node |
| **Combine with** | AND / OR logic |
| **Output** | Single output — only matching items pass through |

---

## If Node

Splits workflow into two branches based on a condition.

| Parameter | Detail |
|-----------|--------|
| **Conditions** | One or more conditions combined with AND/OR |
| **Operators** | Equal, Not Equal, Contains, Regex, Greater/Less than, Is Empty, etc. |
| **Data types** | String, Number, Boolean, Date, Array, Object |
| **Output** | Two outputs — `true` (matching) and `false` (non-matching) |

**Example condition:** `{{ $json.score }}` `greater than` `80`

---

## Switch Node

Routes items to one of many output paths based on a value.

| Parameter | Detail |
|-----------|--------|
| **Mode** | Rules (match conditions) or Expression (return output index) |
| **Rules** | Each rule has a condition and maps to a numbered output |
| **Default** | Items not matching any rule go to the fallback output |
| **Use case** | Route by status, region, type, priority, category |

---

## Merge Node ⚠️

Combines data from two or more incoming branches.

| Mode | Behaviour |
|------|-----------|
| **Append** | Concatenate all items from both inputs |
| **Combine** | Merge items from both inputs into one (by position or key) |
| **SQL Query** | Run a SQL-like query to join items |
| **Choose Branch** | Output items from one specific branch |
| **Wait** | Wait for both branches to complete before merging |
| **⚠️ Warning** | Input order matters — left input is input 1, right is input 2. Reversing them changes output |

---

## Loop Over Items (Split in Batches)

Process items in chunks; ideal for rate-limited APIs.

| Parameter | Detail |
|-----------|--------|
| **Batch size** | Number of items per batch |
| **Output 0** | Items in the current batch (feeds into loop body) |
| **Output 1** | Signals loop completion (connect to post-loop nodes) |

---

## Wait Node

Pause workflow execution temporarily.

| Mode | Detail |
|------|--------|
| **Time Interval** | Pause for N seconds/minutes/hours/days |
| **Specific Time** | Resume at a set date/time |
| **Webhook** | Pause until an external HTTP request resumes it |
| **⚠️ Warning** | Paused executions count toward your execution quota on n8n Cloud |

**Webhook resume URL** is generated automatically — share it with external systems to resume the workflow on demand.

---

## Date & Time Node ⚠️

| Operation | Detail |
|-----------|--------|
| **Format** | Convert date to any string format |
| **Parse** | Parse a date string into a DateTime object |
| **Add / Subtract** | Date arithmetic |
| **Get current time** | Current timestamp in any timezone |
| **Round** | Round to nearest minute/hour/day |
| **⚠️ Warning** | Always specify timezone — default UTC causes off-by-hours bugs in production |

---

## Send Email Node

| Parameter | Detail |
|-----------|--------|
| **Credential** | SMTP (Gmail, Outlook, any SMTP server) |
| **To / CC / BCC** | Static or dynamic from data |
| **Subject** | Static or expression |
| **Body** | Plain text or HTML |
| **Attachments** | Binary data from previous nodes |
| **⚠️ Warning** | Many SMTP servers block emails from automation tools — check spam settings |

---

## Execute Sub-workflow Node

Call another n8n workflow from within a workflow.

| Parameter | Detail |
|-----------|--------|
| **Workflow ID** | Select the target workflow |
| **Source of data** | Input items or fixed parameters |
| **Wait for result** | Synchronous (wait for response) or async (fire and forget) |
| **⚠️ Warning** | Always pass data explicitly — do not rely on global state |

---

## Stop And Error Node

| Parameter | Detail |
|-----------|--------|
| **Error message** | Custom message to surface in execution log |
| **Error object** | Optional structured error data |
| **Use case** | Validation gates — stop if required data is missing or invalid |

---

## SSH Node ⚠️

| Attribute | Detail |
|-----------|--------|
| **Purpose** | Execute commands on remote servers via SSH |
| **Auth** | Password or Private Key |
| **⚠️ Security Risk** | Outputs include command results — never expose sensitive output to downstream untrusted nodes |
| **⚠️ Risk** | Avoid running destructive commands without a Stop And Error safeguard |

---

## Execute Command ⚠️

| Attribute | Detail |
|-----------|--------|
| **Purpose** | Run shell commands on the n8n server |
| **⚠️ Security Risk** | Commands run as the n8n process user — avoid in production unless absolutely necessary |
| **⚠️ Risk** | Never expose this node's inputs to untrusted user data |
| **Alternative** | Use SSH node for remote server commands instead |

---

## Crypto Node

| Operation | Detail |
|-----------|--------|
| **Hash** | SHA-256, SHA-512, MD5 — one-way |
| **HMAC** | Hash-based message authentication (webhook signature validation) |
| **Encrypt** | AES, etc. — two-way with key |
| **⚠️ Common confusion** | Hashing is one-way; encryption is reversible. Choose correctly for your use case |

---

## TOTP Node

| Attribute | Detail |
|-----------|--------|
| **Purpose** | Generate or verify time-based one-time passwords (2FA) |
| **Secret** | Base32-encoded TOTP secret |
| **⚠️ Warning** | Server time must be accurate (within ±30 seconds) — NTP sync required |

---

## Guardrails Node (AI)

| Attribute | Detail |
|-----------|--------|
| **Purpose** | Add safety constraints and validation to AI output |
| **Rules** | Define allowed/blocked patterns for model responses |
| **Use case** | Prevent AI agent from producing harmful or off-topic content |
| **⚠️ Note** | Experimental — test thoroughly before using in production |

---

## Important Node Combinations

| Pattern | Nodes Used | Use Case |
|---------|------------|----------|
| **Conditional branch** | If → (nodes A) + (nodes B) → Merge | Route by status or category |
| **Batch processing** | Loop Over Items → HTTP Request → Loop | Avoid API rate limits |
| **Error handling** | Error Trigger → Slack → Stop | Global alert on any failure |
| **Approval gate** | Wait (webhook) → If → Continue/Stop | Human-in-the-loop approval |
| **Scheduled ETL** | Schedule → HTTP Request → Filter → Google Sheets | Regular data sync |
| **Form-to-CRM** | Form Trigger → HubSpot → Gmail | New lead capture |
| **Sub-workflow fan-out** | Split Out → Execute Sub-workflow | Parallel processing |
| **AI validation** | LLM Chain → Guardrails → If → Continue/Stop | Safe AI output |

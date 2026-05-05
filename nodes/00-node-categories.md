# n8n Node Categories — Complete Index

> Author: Shrinivas Ramaprasad | May 2026

---

## Category Map

| Category | Role | Can Start Workflow? | Primary File |
|---|---|---|---|
| **Trigger Nodes** | Start workflow on event | ✅ Yes | `01-trigger-nodes-complete.md` |
| **Regular / Core Nodes** | Transform, route, combine | ❌ No | `05-node-fields-guide.md` |
| **Code Nodes** | Custom JS/Python logic | ❌ No | This file |
| **Webhook Nodes** | HTTP receive/respond | Trigger can | `02-webhook-deep-dive.md` |
| **HTTP Nodes** | Outbound HTTP calls | ❌ No | `05-node-fields-guide.md` |
| **AI Nodes** | LLM pipelines, agents | ❌ No | `03-ai-nodes-and-open-source-models.md` |
| **App/Integration Nodes** | External service actions | App triggers can | `integrations/` folder |

---

## Category 1 — Trigger Nodes

Full step-by-step guide: `01-trigger-nodes-complete.md`

| Node | Fires When | Credential | Method |
|---|---|---|---|
| Manual Trigger | You click Execute | None | Manual |
| Schedule Trigger | Cron/interval | None | Internal cron |
| Webhook | HTTP request arrives | Optional | Push |
| n8n Form Trigger | Form submitted | None | Push |
| Chat Trigger | Chat message | None | Push |
| Execute Sub-workflow Trigger | Parent calls it | None | Push |
| Error Trigger | Any workflow fails | None | Push |
| Activation Trigger | Workflow toggled | None | Push |
| Gmail Trigger | New email | Google OAuth2 | Polling |
| Slack Trigger | Message/mention | Slack Token | Webhook |
| GitHub Trigger | Push/PR/issue | GitHub PAT | Webhook |
| Telegram Trigger | Bot message | Telegram Token | Polling/Webhook |
| Stripe Trigger | Payment event | Stripe Key | Webhook |
| Postgres Trigger | DB change | PostgreSQL | Push |
| Redis Trigger | Pub/Sub message | Redis | Push |
| Local File Trigger | File change | None | Push (self-hosted) |
| MCP Server Trigger | AI agent calls it | None | Push |

---

## Category 2 — Regular / Core Nodes

### Data Transformation

| Node | Purpose | When to Use |
|---|---|---|
| Edit Fields (Set) | Add/change/remove fields | After every API call to clean/rename fields |
| Filter | Keep matching items only | Remove inactive users, old records |
| Remove Duplicates | Deduplicate by field | Clean email lists, API results |
| Sort | Sort by field | Order records before exporting |
| Limit | Keep first N items | Get top 10 results |
| Split Out | Explode array into items | Process each order/user individually |
| Aggregate | Combine items into one value | Sum amounts, count records |
| Summarize | GROUP BY equivalent | Sales by region, tickets by category |
| Crypto | Hash, HMAC, encrypt | Validate Stripe/GitHub signatures |
| Date & Time | Parse, format, calculate | Add 30 days, format for emails |
| Convert to File | Data → CSV/XLSX/PDF | Export spreadsheets |
| Extract From File | CSV/PDF → structured data | Read uploads |
| JWT | Sign/verify tokens | API auth |
| AI Transform | LLM-powered transformation | "Translate this", "Summarise this" |

### Flow Control

| Node | Purpose | Sample Use |
|---|---|---|
| If | Binary true/false branch | Route high-value orders |
| Switch | Multi-path routing | Route by country/priority/status |
| Merge | Combine parallel branches | Join user data + order data |
| Loop Over Items | Process in batches | Rate-limited API, 10 items at a time |
| Wait | Pause: time/webhook | Approval flow, retry delay |
| Stop and Error | Halt with message | Validation gate |
| Execute Sub-workflow | Call another workflow | Reusable send-notification logic |

---

## Category 3 — Code Nodes

| Node | Language | Purpose |
|---|---|---|
| Code | JavaScript or Python | Any custom logic: transform, filter, calculate |
| AI Transform | Natural language prompt | Simple LLM transformations without code |
| LangChain Code | JavaScript | Custom LangChain pipeline |

### Code Node Modes

| Mode | When to Use | Access Data |
|---|---|---|
| Run Once for All Items | Aggregate, group, build CSV | `$input.all()` |
| Run Once for Each Item | Transform each item | `$json` |

### Required Return Format

```javascript
// Always return array of objects with 'json' key
return [{ json: { field: 'value', count: 42 } }];

// Multiple items:
return items.map(i => ({ json: { ...i.json, processed: true } }));
```

### Built-in JavaScript Variables

| Variable | Returns | Sample |
|---|---|---|
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
|---|---|---|
| `return { name: 'x' }` | "Data is not an array" | `return [{ json: { name: 'x' } }]` |
| `return []` | Empty — downstream gets nothing | Return `[{ json: {} }]` if needed |
| `$json.user.name` when user is null | TypeError | `$json.user?.name ?? 'Unknown'` |

---

## Category 4 — Webhook Nodes

Full guide: `02-webhook-deep-dive.md`

| Node | Direction | Purpose |
|---|---|---|
| Webhook (trigger) | Inbound | Receive HTTP requests → start workflow |
| Respond to Webhook | Outbound | Send custom HTTP response |

| Caller | Recommended Mode |
|---|---|
| Stripe, GitHub, Shopify | Immediately (don't need response data) |
| Your own frontend app | Last Node (wait for result) |
| Need custom status code | Respond to Webhook Node |

---

## Category 5 — HTTP Nodes

| Node | Purpose |
|---|---|
| HTTP Request | Call any REST/GraphQL/SOAP API |
| GraphQL | Execute GraphQL queries |
| RSS Read | Fetch and parse RSS/Atom feeds |

### HTTP Request Key Fields

| Field | Options | Default | If Misconfigured |
|---|---|---|---|
| Method | GET/POST/PUT/PATCH/DELETE | — | 405 error |
| URL | Static or `{{ expression }}` | — | Fails entirely |
| Authentication | None/Predefined/Generic | None | 401 Unauthorized |
| Body Type | JSON/Form/Binary | JSON | API rejects |
| Timeout | milliseconds | 300000 | ETIMEDOUT |
| Retry On Fail | ON + count + delay | OFF | Single fail kills run |
| Pagination | Offset/Cursor/Link Header | — | First page only |

---

## Category 6 — AI Nodes

Full guide: `03-ai-nodes-and-open-source-models.md`

### Root Nodes (Orchestrators)

| Node | Required Sub-nodes |
|---|---|
| AI Agent | Chat Model (required) + Tools + Memory |
| Basic LLM Chain | Chat Model |
| Q&A Chain | Chat Model + Vector Store Retriever |
| Summarization Chain | Chat Model |
| Information Extractor | Chat Model |
| Text Classifier | Chat Model |

### Sub-node Types

| Type | Examples |
|---|---|
| Chat Models | OpenAI, Anthropic, Gemini, Ollama, Groq, DeepSeek |
| Memory | Simple (testing), Redis (production), Postgres |
| Tools | HTTP Request, Calculator, Wikipedia, Custom Code |
| Vector Stores | Pinecone, PGVector, Supabase, Chroma, Qdrant |

---

## Category 7 — App / Integration Nodes

| Service | Key Operations |
|---|---|
| Google Sheets | Read/write/append/update rows |
| Gmail | Send HTML email, read/filter/label |
| Google Drive | Upload, download (PDF/XLSX), search |
| Google Calendar | Events, Google Meet, attendees |
| Telegram | Send messages/files; Send and Wait |
| Slack | Messages, channels, file upload |
| MySQL | Query, insert, update, delete |
| PostgreSQL | Query, insert, update, upsert |
| MongoDB | Document CRUD |
| Redis | Key-value, pub/sub |
| HubSpot | Contacts, deals, companies |
| Stripe | Customers, payments, subscriptions |
| GitHub | Files, issues, PRs, releases |

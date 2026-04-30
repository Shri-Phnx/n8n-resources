# n8n Nodes — Complete Overview & Master Index

> **Author:** Shrinivas Ramaprasad  
> **Last updated:** April 2026  
> **Source:** [n8n Docs](https://docs.n8n.io/integrations/)

---

## What is a Node?

A **node** is a single processing unit in an n8n workflow. Every node:
- Receives one or more **input items** (JSON objects)
- Performs a defined action (API call, logic, data transform, file op, etc.)
- Outputs one or more **result items** to the next node

Nodes connect via **edges/connectors** to form a directed workflow graph. Data flows left → right through the chain.

---

## Node Type Categories

| Type | Purpose | Self-starting? |
|------|---------|----------------|
| **Trigger** | Start a workflow in response to an event | ✅ Yes |
| **Core** | Built-in logic, transformation, HTTP, flow control | ❌ No |
| **App / Action** | Interact with 300+ external services | ❌ No |
| **AI / Cluster** | Multi-node AI agent pipelines (LLMs, RAG, tools) | ❌ No |
| **Community** | Third-party nodes from npm (verified or unverified) | ❌ No |
| **Custom** | Nodes you build yourself in TypeScript | ❌ No |

---

## How to Find & Add Nodes

### Method 1 — Keyboard shortcut (fastest)
```
Press N → search field opens → type node name → press Enter
```

### Method 2 — Canvas `+` button
Click the **`+`** icon in the top-right corner of the canvas

### Method 3 — Node connector button
Click the **`+`** icon on the right edge of any existing node on the canvas

### Method 4 — Command bar
```
Ctrl / Cmd + K → type node name → select from list
```

---

## Node Panel Categories

After your trigger node is added, the panel groups nodes into:

| Category | Contents |
|----------|----------|
| **Advanced AI** | LLM chains, AI agents, vector stores, embeddings |
| **Actions in an App** | All 300+ service integrations |
| **Data transformation** | Edit Fields, Code, Merge, Sort, Filter, Aggregate |
| **Flow** | If, Switch, Loop, Wait, Error Trigger |
| **Core** | HTTP Request, Webhook, Schedule, Email, etc. |
| **Human in the loop** | Wait-for-approval and human interaction patterns |

---

## Master Node Index

### Trigger Nodes

| Node | Trigger Event | Primary Use Case |
|------|--------------|------------------|
| **Manual Trigger** | Button click in UI | Testing, one-off runs, demos |
| **Schedule Trigger** | Cron / interval | Daily reports, regular data sync |
| **Webhook** | Incoming HTTP POST/GET/etc. | Real-time integrations, external app events |
| **Email Trigger (IMAP)** | New email arrives | Email-to-workflow automation |
| **n8n Form Trigger** | Form submission | Lead capture, internal data collection |
| **Chat Trigger** | Chat message received | AI chatbot and agent UIs |
| **Activation Trigger** | Workflow activated / deactivated | Bootstrap actions, setup routines |
| **Error Trigger** | Execution error in any workflow | Global error handling and alerting |
| **SSE Trigger** | Server-Sent Events stream | Real-time data pipelines |
| **Local File Trigger** | File system change | Local file monitoring (self-hosted only) |
| **RSS Feed Trigger** | New RSS/Atom item | Blog, news, and content monitoring |
| **Execute Sub-workflow Trigger** | Called by parent workflow | Sub-workflow / function entry points |
| **MCP Server Trigger** | MCP protocol call | n8n as an AI agent tool server |
| **n8n Trigger** | n8n instance events | Internal platform event responses |
| **Workflow Trigger** | Another workflow event | Cross-workflow event chaining |
| **GitHub Trigger** | Push, PR, issue, release events | CI/CD, DevOps automation |
| **Gmail Trigger** | New Gmail message matching filter | Email processing pipelines |
| **Slack Trigger** | Slack events (message, reaction, etc.) | Slack bot workflows |
| **Airtable Trigger** | Record created / updated | Database-driven automation |
| **Stripe Trigger** | Payment, subscription, customer events | E-commerce automation |
| **Shopify Trigger** | Order, product, customer events | E-commerce order processing |
| **Jira Trigger** | Issue created / updated | Project management automation |
| **Notion Trigger** | Database item changes | Notion-based workflows |
| **Typeform Trigger** | Form submission | Survey and lead form processing |
| **HubSpot Trigger** | CRM events | CRM-driven workflows |
| **Salesforce Trigger** | Object create / update | Enterprise CRM automation |
| **Telegram Trigger** | Bot message received | Telegram bot workflows |
| **Discord Trigger** | Server events | Discord bot automation |
| **Linear Trigger** | Issue events | Engineering team workflows |
| **Postgres Trigger** | Database changes (LISTEN/NOTIFY) | DB-driven real-time workflows |
| **Redis Trigger** | Pub/Sub messages | Event-driven microservice patterns |
| **Kafka Trigger** | Kafka topic messages | High-volume event streaming |
| **RabbitMQ Trigger** | Queue messages | Message queue processing |
| **MQTT Trigger** | IoT / MQTT messages | IoT sensor and device workflows |
| **Google Sheets Trigger** | Spreadsheet changes | Spreadsheet-driven automation |
| **Google Calendar Trigger** | Event create / update | Calendar-based reminders |
| **Webhook (100+ services)** | Service-specific webhooks | Shopify, Stripe, Jira, GitHub, etc. |

---

### Core Nodes — Data Transformation

| Node | Purpose |
|------|---------|
| **Edit Fields (Set)** | Add, update, rename, or remove fields on items |
| **Code** | Write custom JavaScript or Python for any transformation |
| **Filter** | Keep only items that match a condition (discard the rest) |
| **Rename Keys** | Rename JSON property keys without touching values |
| **Aggregate** | Aggregate multiple items into one (sum, list, count, etc.) |
| **Sort** | Sort items by one or more fields |
| **Limit** | Keep only the first N items |
| **Split Out** | Explode an array field into individual items |
| **Summarize** | Group items and aggregate like SQL GROUP BY |
| **Remove Duplicates** | Deduplicate items by one or more key fields |
| **AI Transform** | Transform data using a natural language instruction + LLM |
| **HTML** | Extract data from HTML pages or generate HTML output |
| **XML** | Parse or generate XML data |
| **Markdown** | Convert between Markdown and HTML |
| **Crypto** | Hash (SHA, MD5), encrypt, and decrypt data |
| **Date & Time** | Parse, format, calculate, and manipulate date/time values |
| **Convert to File** | Convert data to CSV, XLSX, PDF, or other file formats |
| **Extract From File** | Extract structured data from uploaded files |
| **Compression** | Zip or unzip files and folders |
| **JWT** | Sign and verify JSON Web Tokens |
| **TOTP** | Generate time-based one-time passwords (2FA codes) |

---

### Core Nodes — Flow Control

| Node | Purpose |
|------|---------|
| **If** | Binary branch — routes items to true or false output |
| **Switch** | Multi-path routing — routes items based on a value match |
| **Merge** | Combine data from two or more parallel branches |
| **Loop Over Items (Split in Batches)** | Process items in batches; iterate until complete |
| **Wait** | Pause execution — resume after time delay or webhook signal |
| **Stop And Error** | Halt execution and throw a custom error message |
| **No Operation** | Pass-through placeholder — useful for workflow planning |
| **Execute Sub-workflow** | Call another n8n workflow (synchronous or async) |

---

### Core Nodes — HTTP & Communication

| Node | Purpose |
|------|---------|
| **HTTP Request** | Make any HTTP/REST/GraphQL API call with full control |
| **Webhook** | Receive and respond to HTTP requests mid-workflow |
| **Respond to Webhook** | Send a custom HTTP response back to a webhook caller |
| **Send Email** | Send emails via SMTP (Gmail, Outlook, any SMTP server) |
| **GraphQL** | Execute GraphQL queries and mutations |
| **RSS Read** | Fetch and parse RSS/Atom feeds |
| **FTP** | Transfer files via FTP or SFTP |
| **SSH** | Run commands on remote servers via SSH |
| **Read/Write Files from Disk** | Read and write local files (self-hosted instances only) |

---

### Core Nodes — AI & Utilities

| Node | Purpose |
|------|---------|
| **AI Transform** | Use an LLM to transform data with a natural language prompt |
| **Evaluation** | Score and evaluate AI model outputs |
| **Evaluation Trigger** | Trigger evaluation runs |
| **Debug Helper** | Inject mock data to test downstream nodes |
| **n8n** | Manage n8n itself — workflows, executions, credentials |
| **Data table** | Display data in a structured table view |
| **MCP Client** | Connect to external MCP (Model Context Protocol) servers |

---

### App / Action Nodes (Key Integrations)

#### Productivity & Communication
| App | Key Actions |
|-----|-------------|
| **Slack** | Send message, post to channel, create channel, manage users |
| **Gmail** | Send/read emails, manage labels, create drafts |
| **Microsoft Outlook** | Email, calendar events, contacts |
| **Microsoft Teams** | Send message, manage channels and meetings |
| **Telegram** | Send message/photo/file, manage bot |
| **Discord** | Send message, manage server and channels |
| **Notion** | Create/update pages and databases |
| **Google Calendar** | Create, update, delete events |
| **Microsoft To Do** | Create and manage tasks |
| **Todoist** | Manage tasks, projects, comments |
| **Zoom** | Create meetings, manage webinars |

#### CRM & Sales
| App | Key Actions |
|-----|-------------|
| **HubSpot** | Contacts, deals, companies, email sequences |
| **Salesforce** | Leads, contacts, opportunities, reports |
| **Pipedrive** | Deals, persons, activities, organisations |
| **Zoho CRM** | Contacts, leads, modules |
| **ActiveCampaign** | Contacts, campaigns, tags, automations |
| **Freshdesk** | Tickets, contacts, agents |
| **Freshservice** | Incidents, service requests, assets |
| **Copper** | Leads, people, companies |

#### Data & Databases
| App | Key Actions |
|-----|-------------|
| **Google Sheets** | Read/write rows, create/update sheets |
| **Airtable** | Records, fields, views, formulas |
| **Postgres** | SQL queries, insert, update, delete |
| **MySQL** | SQL queries, transactions |
| **MongoDB** | Documents, collections, aggregations |
| **Redis** | Key-value get/set, pub/sub |
| **Supabase** | Tables, auth, storage, realtime |
| **NocoDB** | Tables, records, views |
| **Baserow** | Tables and row operations |

#### File Storage & Documents
| App | Key Actions |
|-----|-------------|
| **Google Drive** | Upload, download, move, share files |
| **Dropbox** | Files, folders, sharing, paper docs |
| **Microsoft OneDrive** | Files, folders, permissions |
| **AWS S3** | Object storage, presigned URLs, bucket management |
| **Box** | Files, folders, comments, tasks |
| **Nextcloud** | Files, shares, activities |

#### Developer & DevOps
| App | Key Actions |
|-----|-------------|
| **GitHub** | Repos, issues, PRs, commits, releases |
| **GitLab** | Projects, pipelines, merge requests |
| **Jira** | Issues, projects, sprints, comments |
| **Linear** | Issues, cycles, teams, labels |
| **Vercel** | Deployments, projects, domains |
| **AWS Lambda** | Invoke functions, manage functions |
| **Cloudflare** | DNS, workers, zones, KV store |
| **Jenkins** | Build jobs, queue builds |
| **CircleCI** | Pipelines, workflows, jobs |

#### AI & ML Services
| App | Key Actions |
|-----|-------------|
| **OpenAI** | Chat, images (DALL-E), audio, assistants, files |
| **Anthropic** | Claude chat completions (all models) |
| **Google Gemini** | Chat, multimodal, embeddings |
| **Mistral AI** | Chat completions |
| **Perplexity** | Search-augmented completions |
| **DeepSeek** | Chat completions (R1, V3) |
| **Hugging Face** | Model inference via API |
| **xAI** | Grok model completions |

#### Payments & E-commerce
| App | Key Actions |
|-----|-------------|
| **Stripe** | Customers, payments, subscriptions, refunds |
| **Shopify** | Products, orders, customers, inventory |
| **WooCommerce** | Products, orders, customers |
| **PayPal** | Payments, subscriptions, invoices |
| **Chargebee** | Subscriptions, invoices, customers |

---

### AI / Cluster Nodes

See [`03-ai-nodes-and-nvidia.md`](./03-ai-nodes-and-nvidia.md) for full details and NVIDIA NIM setup.

| Category | Examples |
|----------|----------|
| **Root (orchestrators)** | AI Agent, Basic LLM Chain, Q&A Chain, Summarization Chain |
| **LLM Models** | OpenAI, Anthropic, Gemini, Mistral, Ollama, DeepSeek, Groq |
| **Memory** | Simple Memory, Redis Memory, Postgres Memory |
| **Tools** | HTTP Request Tool, Calculator, Wikipedia, Workflow Tool |
| **Vector Stores** | Pinecone, PGVector, Supabase, Chroma, Qdrant, Weaviate |
| **Embeddings** | OpenAI, Gemini, Ollama, HuggingFace, Mistral |

---

## Related Documents

| Document | Description |
|----------|-------------|
| [`01-core-and-trigger-nodes.md`](./01-core-and-trigger-nodes.md) | Core and trigger node details with parameters |
| [`02-webhook-deep-dive.md`](./02-webhook-deep-dive.md) | Complete Webhook node guide |
| [`03-ai-nodes-and-nvidia.md`](./03-ai-nodes-and-nvidia.md) | AI cluster nodes and NVIDIA NIM integration |
| [`04-community-and-custom-nodes.md`](./04-community-and-custom-nodes.md) | Community nodes and custom node development |

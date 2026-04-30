# n8n Nodes — Complete Overview & Master Index

> **Author:** Shrinivas Ramaprasad  
> **Last updated:** April 2026  
> **Source:** n8n Official Docs + N8N_All_Nodes.xlsx (819 nodes verified)

---

## What is a Node?

A **node** is a single processing unit in an n8n workflow. Every node:
- Receives one or more **input items** (JSON objects)
- Performs a defined action (API call, logic, data transform, file op, etc.)
- Outputs one or more **result items** to the next node

Nodes connect via **edges/connectors** to form a directed workflow graph. Data flows left → right through the chain.

---

## Node Type Summary (from verified Excel dataset)

| Type | Count | Purpose | Self-starting? |
|------|-------|---------|----------------|
| **Core Nodes** | 67 | Built-in logic, transformation, HTTP, flow control | Triggers: ✅ / Actions: ❌ |
| **Actions** | 270 | Interact with 270+ external services | ❌ No |
| **Triggers** | 90 | Start workflows from external events | ✅ Yes |
| **Root Nodes (AI)** | 21 | AI agent pipelines, chains, vector stores | ❌ No |
| **Sub-nodes (AI)** | 63 | LLMs, memory, tools, embeddings, parsers | ❌ No (attach to Root) |
| **Credentials** | 308 | Authentication configs for all integrations | — |
| **Community** | Growing | Third-party npm packages | Varies |
| **Custom** | Build your own | TypeScript nodes you develop | Varies |

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
| **Actions in an App** | All 270+ service integrations |
| **Data transformation** | Edit Fields, Code, Merge, Sort, Filter, Aggregate |
| **Flow** | If, Switch, Loop, Wait, Error Trigger |
| **Core** | HTTP Request, Webhook, Schedule, Email, etc. |
| **Human in the loop** | Wait-for-approval and human interaction patterns |

---

## Core Nodes — Complete List (67 nodes)

### Trigger Nodes (Core)

| Node | Purpose | Key Properties | Common Use Case | ⚠️ Notes |
|------|---------|----------------|-----------------|----------|
| **Activation Trigger** | Starts workflow when activated | None | Auto-run workflows | Only works when workflow is active |
| **Chat Trigger** | Trigger via chat input | Chat source | Chatbots | Needs integration setup |
| **Email Trigger (IMAP)** | Trigger on new email | Mailbox, filters | Email automation | Polling delays |
| **Error Trigger** | Runs on workflow error | None | Error workflows | Runs only on failure |
| **Evaluation Trigger** | Trigger based on evaluation | Expression | Conditional automation | Can loop if misused ⚠️ |
| **Execute Sub-workflow Trigger** | Trigger from parent workflow | None | Workflow chaining | Used with parent |
| **Local File Trigger** | Trigger on file changes | Folder path | File automation | Depends on server FS |
| **Manual Trigger** | Start workflow manually | None | Testing | Not for production |
| **MCP Server Trigger** | Trigger via MCP protocol | None | AI workflows | Needs MCP setup |
| **n8n Form Trigger** | Trigger from form submit | None | Data collection | Needs Form node |
| **n8n Trigger** | Trigger from n8n events | Event type | System events | Limited triggers |
| **RSS Feed Trigger** | Trigger on new RSS items | Feed URL | Monitoring feeds | Delay depends on polling |
| **Schedule Trigger** | Run on schedule | Cron expression | Scheduled tasks | Timezone issues ⚠️ |
| **SSE Trigger** | Trigger via server events | Endpoint | Real-time updates | Needs SSE source |
| **Webhook** | Receive HTTP requests | URL, method | API endpoints | Public exposure ⚠️ |
| **Workflow Trigger** | Trigger from another workflow | Workflow ID | Workflow chaining | Avoid loops ⚠️ |

---

### Data Transformation Nodes

| Node | Purpose | Key Properties | Common Use Case | ⚠️ Notes |
|------|---------|----------------|-----------------|----------|
| **Aggregate** | Combine multiple items into one | Fields, Aggregation type | Summarizing data | Requires grouped input |
| **AI Transform** | Use AI to modify data | Prompt, Model | AI enrichment | Watch token usage ⚠️ |
| **Compare Datasets** | Compare two datasets | Fields to compare | Data validation | Input order matters |
| **Edit Fields (Set)** | Add/edit/remove fields | Fields, Keep Only Set | Data mapping | "Keep Only Set" can delete existing data ⚠️ |
| **Filter** | Filter items by condition | Conditions | Data filtering | Removes unmatched items |
| **HTML** | Extract or generate HTML | Selectors | Web scraping | CSS selectors needed |
| **Limit** | Restrict number of items | Limit number | Performance control | Cuts data permanently |
| **Loop Over Items (Split in Batches)** | Process items in batches | Batch size | Large datasets | Prevent memory issues |
| **Merge** | Combine multiple inputs | Mode (append, merge) | Join data | Input order matters ⚠️ |
| **Remove Duplicates** | Eliminate duplicate items | Fields | Data cleaning | Needs correct key field |
| **Rename Keys** | Rename JSON fields | Mapping | Data transformation | Case-sensitive |
| **Sort** | Sort data | Field, order | Data ordering | Only affects current output |
| **Split Out** | Split array into individual items | Field | Normalize data | Creates multiple items |
| **Summarize** | Group and aggregate (SQL-like) | Fields, operations | Reports/analytics | Like SQL GROUP BY |
| **XML** | Parse or create XML | Input/output format | Data conversion | Structure-sensitive |

---

### Flow Control Nodes

| Node | Purpose | Key Properties | Common Use Case | ⚠️ Notes |
|------|---------|----------------|-----------------|----------|
| **Evaluation** | Evaluate expressions | Expression | Conditional checks | Use expressions carefully |
| **Execute Sub-workflow** | Call another workflow | Workflow ID | Modular design | Pass data explicitly |
| **If** | Conditional branching | Conditions | Logic flow | Only 2 outputs |
| **Merge** | Combine parallel branches | Mode | Join data streams | |
| **No Operation** | Pass-through placeholder | None | Placeholder | Useful for testing/planning |
| **Stop And Error** | Stop workflow with error | Message | Fail handling/validation | |
| **Switch** | Multiple conditional branches | Cases | Complex routing | More flexible than IF |
| **Wait** | Delay/pause workflow | Time or webhook | Scheduling, approvals | Can pause indefinitely ⚠️ |

---

### HTTP & Integration Nodes

| Node | Purpose | Key Properties | Common Use Case | ⚠️ Notes |
|------|---------|----------------|-----------------|----------|
| **FTP** | Transfer files via FTP/SFTP | Host, credentials | File sync | Passive mode issues |
| **GraphQL** | Query GraphQL APIs | Query, endpoint | API integration | Requires schema knowledge |
| **HTTP Request** | Call external APIs | URL, Method, Headers | Integrations | Handle pagination ⚠️ |
| **MCP Client** | Connect to MCP server | Endpoint | AI tools | Advanced usage |
| **n8n** | Access n8n internal API | Operation | Manage workflows | Requires permissions |
| **Respond to Webhook** | Send webhook response | Response data | API endpoints | Must match request |
| **RSS Read** | Read RSS feeds | URL | News feeds | Polling-based |
| **SSH** | Execute commands remotely | Host, key | Server automation | Security risk ⚠️ |
| **Webhook** | Receive HTTP requests | URL, method | API endpoints | See 02-webhook-deep-dive.md |

---

### Security Nodes

| Node | Purpose | Common Use Case | ⚠️ Notes |
|------|---------|-----------------|----------|
| **Crypto** | Encrypt/decrypt/hash data | Security tasks | Hash vs encrypt confusion |
| **JWT** | Create/verify tokens | Auth systems | Expiry handling |
| **LDAP** | Query directory services | User lookup | Complex setup |
| **TOTP** | Generate/verify OTP codes | 2FA systems | Time sync required |

---

### File Handling Nodes

| Node | Purpose | Common Use Case | ⚠️ Notes |
|------|---------|-----------------|----------|
| **Compression** | Compress/decompress files | File handling | Binary data required |
| **Convert to File** | Convert data to file format | Export data | Large data may slow |
| **Edit Image** | Modify images | Image processing | Needs binary input |
| **Extract From File** | Extract data from files | CSV/JSON parsing | File must be valid |
| **Read/Write Files from Disk** | Handle local files | File storage | Server access needed |

---

### Developer & AI Utility Nodes

| Node | Purpose | Common Use Case | ⚠️ Notes |
|------|---------|-----------------|----------|
| **Code** | Run custom JavaScript or Python | Complex transformations | Runs per item unless specified |
| **Data Table** | Store structured data | Internal DB | Limited scalability |
| **Date & Time** | Format or manipulate dates | Scheduling logic | Timezones important ⚠️ |
| **Debug Helper** | Inspect data | Debugging | Remove in production |
| **Execute Command** | Run shell commands | Server automation | Security risk ⚠️ |
| **Execution Data** | Access past execution data | Logging/debugging | Limited retention |
| **Git** | Interact with Git repos | Dev workflows | Auth setup needed |
| **Guardrails** | Add AI safety constraints | AI validation | Prevents bad outputs |
| **Markdown** | Convert markdown content | Formatting | HTML output |
| **n8n Form** | Create web forms | User input | UI-based |
| **Send Email** | Send emails via SMTP | Notifications | Watch spam filters |

---

## App / Action Nodes (270 integrations)

### Productivity & Communication
| App | Key Actions |
|-----|-------------|
| **Slack** | Send message, post to channel, create channel |
| **Gmail** | Send/read emails, manage labels, create drafts |
| **Microsoft Outlook** | Email, calendar events, contacts |
| **Microsoft Teams** | Send message, manage channels |
| **Telegram** | Send message/photo/file, manage bot |
| **Discord** | Send message, manage server |
| **Notion** | Create/update pages and databases |
| **Google Calendar** | Create, update, delete events |
| **Todoist** | Tasks, projects, comments |
| **Zoom** | Create meetings, manage webinars |

### CRM & Sales
| App | Key Actions |
|-----|-------------|
| **HubSpot** | Contacts, deals, companies, email sequences |
| **Salesforce** | Leads, contacts, opportunities |
| **Pipedrive** | Deals, persons, activities |
| **ActiveCampaign** | Contacts, campaigns, tags |
| **Freshdesk / Freshservice** | Tickets, contacts, assets |

### Data & Databases
| App | Key Actions |
|-----|-------------|
| **Google Sheets** | Read/write rows, create/update sheets |
| **Airtable** | Records, fields, views |
| **Postgres** | SQL queries, insert, update, delete |
| **MySQL** | SQL queries, transactions |
| **MongoDB** | Documents, collections, aggregations |
| **Redis** | Key-value get/set, pub/sub |
| **Supabase** | Tables, auth, storage |

### Developer & DevOps
| App | Key Actions |
|-----|-------------|
| **GitHub** | Repos, issues, PRs, commits |
| **GitLab** | Projects, pipelines, merge requests |
| **Jira** | Issues, projects, sprints |
| **Linear** | Issues, cycles, teams |
| **Vercel** | Deployments, projects, domains |
| **AWS Lambda** | Invoke functions |

### AI & ML Services
| App | Key Actions |
|-----|-------------|
| **OpenAI** | Chat, images, audio, assistants |
| **Anthropic** | Claude chat completions |
| **Google Gemini** | Chat, multimodal, embeddings |
| **Mistral AI** | Chat completions |
| **DeepSeek** | Chat completions (R1, V3) |
| **Hugging Face** | Model inference via API |
| **xAI (Grok)** | Grok model completions |

---

## All 90 Trigger Nodes

| Trigger Node | Category | Purpose | ⚠️ Notes |
|-------------|----------|---------|----------|
| ActiveCampaign Trigger | CRM & Marketing | Trigger on contact/campaign events | API limits |
| Acuity Scheduling Trigger | Scheduling | Trigger on appointments | Delay due to polling |
| Affinity Trigger | CRM & Sales | Trigger on CRM updates | API limits |
| Airtable Trigger | Database | Trigger on record changes | Rate limits ⚠️ |
| AMQP Trigger | Messaging | Listen to queue messages | Needs broker |
| Asana Trigger | Productivity | Trigger on task updates | API limits |
| Autopilot Trigger | CRM & Marketing | Trigger on customer journeys | Event-based |
| AWS SNS Trigger | Messaging | Trigger on notifications | Subscription required |
| Bitbucket Trigger | DevOps | Trigger on repo events | Webhook setup |
| Box Trigger | Cloud Storage | Trigger on file changes | Delay possible |
| Brevo Trigger | CRM & Marketing | Trigger on email events | API limits |
| Cal Trigger | Scheduling | Trigger on scheduling events | Similar to Calendly |
| Calendly Trigger | Scheduling | Trigger on bookings | Delay possible |
| Chargebee Trigger | Finance | Trigger on subscription events | Webhooks needed |
| ClickUp Trigger | Productivity | Trigger on task changes | API limits |
| Clockify Trigger | Productivity | Trigger on time entries | Limited API |
| ConvertKit Trigger | CRM & Marketing | Trigger on subscriber actions | Tag-based |
| Copper Trigger | CRM & Sales | Trigger on CRM updates | API limits |
| crowd.dev Trigger | Analytics | Trigger on community events | API limits |
| Customer.io Trigger | CRM & Marketing | Trigger on customer actions | Event-driven |
| Emelia Trigger | CRM & Marketing | Trigger on outreach events | Rate limits |
| Eventbrite Trigger | Events | Trigger on event activity | Delay possible |
| Facebook Lead Ads Trigger | Marketing | Trigger on new leads | Token expiry ⚠️ |
| Facebook Trigger | Social Media | Trigger on page activity | API limits |
| Figma Trigger (Beta) | Design | Trigger on design updates | Beta limitations ⚠️ |
| Form.io Trigger | Forms | Trigger on form submission | Delay possible |
| Formstack Trigger | Forms | Trigger on submissions | API limits |
| GetResponse Trigger | CRM & Marketing | Trigger on campaign events | Rate limits |
| GitHub Trigger | DevOps | Trigger on repo activity | Webhook setup |
| GitLab Trigger | DevOps | Trigger on repo events | Permissions |
| Gmail Trigger | Communication | Trigger on new emails | Delay ⚠️ |
| Google Business Profile Trigger | Marketing | Trigger on business updates | Permissions |
| Google Calendar Trigger | Productivity | Trigger on events | Timezone issues ⚠️ |
| Google Drive Trigger | Cloud Storage | Trigger on file changes | Delay possible |
| Google Sheets Trigger | Productivity | Trigger on row changes | Rate limits |
| Gumroad Trigger | E-commerce | Trigger on sales | API limits |
| Help Scout Trigger | Support | Trigger on conversations | API limits |
| HubSpot Trigger | CRM & Sales | Trigger on CRM events | Rate limits |
| Invoice Ninja Trigger | Finance | Trigger on invoices | API limits |
| Jira Trigger | Productivity | Trigger on issue updates | API limits |
| Jotform Trigger | Forms | Trigger on submissions | Delay possible |
| Kafka Trigger | Messaging | Consume streaming events | Complex setup ⚠️ |
| Keap Trigger | CRM & Marketing | Trigger on CRM actions | API limits |
| KoboToolbox Trigger | Data Collection | Trigger on submissions | Delay possible |
| Lemlist Trigger | CRM & Marketing | Trigger on outreach activity | Rate limits |
| Linear Trigger | Productivity | Trigger on issues | API limits |
| Mailchimp Trigger | CRM & Marketing | Trigger on subscriber activity | Rate limits |
| MailerLite Trigger | CRM & Marketing | Trigger on email events | API limits |
| Mailjet Trigger | Communication | Trigger on email activity | API limits |
| Mautic Trigger | CRM & Marketing | Trigger on campaigns | Self-hosted required ⚠️ |
| Microsoft OneDrive Trigger | Cloud Storage | Trigger on file updates | Delay possible |
| Microsoft Outlook Trigger | Communication | Trigger on email events | Delay ⚠️ |
| Microsoft Teams Trigger | Communication | Trigger on messages | Permissions |
| MQTT Trigger | IoT | Subscribe to messages | Broker required |
| Netlify Trigger | DevOps | Trigger on deploys | API limits |
| Notion Trigger | Productivity | Trigger on page updates | Delay possible |
| Onfleet Trigger | Logistics | Trigger on delivery updates | API limits |
| PayPal Trigger | Finance | Trigger on transactions | Webhooks ⚠️ |
| Pipedrive Trigger | CRM & Sales | Trigger on deals | API limits |
| Postgres Trigger | Database | Trigger on DB changes | Needs DB config |
| Postmark Trigger | Communication | Trigger on email activity | API limits |
| Pushcut Trigger | Automation | Trigger iOS workflows | iOS only ⚠️ |
| RabbitMQ Trigger | Messaging | Consume queue messages | Ack handling ⚠️ |
| Redis Trigger | Database | Trigger on key changes | Limited triggers |
| Salesforce Trigger | CRM & Sales | Trigger on CRM updates | API limits |
| SeaTable Trigger | Database | Trigger on table changes | Delay possible |
| Shopify Trigger | E-commerce | Trigger on store events | API limits |
| Slack Trigger | Communication | Trigger on messages | Permissions |
| Strava Trigger | Fitness | Trigger on activity | API limits |
| Stripe Trigger | Finance | Trigger on payments | Webhooks ⚠️ |
| SurveyMonkey Trigger | Forms | Trigger on survey responses | Delay possible |
| Taiga Trigger | Productivity | Trigger on tasks/issues | API limits |
| Telegram Trigger | Communication | Trigger on messages | Setup needed ⚠️ |
| TheHive 5 Trigger | Security | Trigger on incidents | Complex setup |
| TheHive Trigger | Security | Trigger on alerts | Complex setup |
| Toggl Trigger | Productivity | Trigger on time tracking | API limits |
| Trello Trigger | Productivity | Trigger on card updates | API limits |
| Twilio Trigger | Communication | Trigger on SMS/calls | Webhook setup ⚠️ |
| Typeform Trigger | Forms | Trigger on submissions | Delay possible |
| Venafi TLS Protect Cloud Trigger | Security | Trigger on cert events | Enterprise tool |
| Webex by Cisco Trigger | Communication | Trigger on meetings/messages | Permissions |
| Webflow Trigger | CMS | Trigger on content updates | API limits |
| WhatsApp Trigger | Communication | Trigger on messages | Meta approval ⚠️ |
| Wise Trigger | Finance | Trigger on transactions | API limits |
| WooCommerce Trigger | E-commerce | Trigger on orders | API limits |
| Workable Trigger | HR | Trigger on hiring events | API limits |
| Wufoo Trigger | Forms | Trigger on submissions | Delay possible |
| Zendesk Trigger | Support | Trigger on tickets | API limits |

---

## AI / Root Nodes (21 nodes)

See [`03-ai-nodes-and-open-source-models.md`](./03-ai-nodes-and-open-source-models.md) for full details.

| Node | Category | Purpose |
|------|----------|---------|
| **AI Agent** | AI & ML | Autonomous AI agent with tools and memory |
| **Basic LLM Chain** | AI & ML | Simple LLM prompt-response flow |
| **Question and Answer Chain** | AI & ML | RAG-based Q&A using context documents |
| **Summarization Chain** | AI & ML | Summarize long text via Map-Reduce |
| **Information Extractor** | AI & ML | Extract structured fields from unstructured text |
| **Text Classifier** | AI & ML | Classify text into predefined categories |
| **Sentiment Analysis** | AI & ML | Detect positive/negative/neutral sentiment |
| **LangChain Code** | AI & ML | Custom LangChain logic in code |
| **Microsoft Agent 365 Trigger** | Trigger/AI | Trigger AI agent from Microsoft ecosystem |
| **Azure AI Search Vector Store** | Vector Database | Store/search embeddings in Azure |
| **Chroma Vector Store** | Vector Database | Lightweight local/hosted vector DB |
| **Milvus Vector Store** | Vector Database | Scalable distributed vector DB |
| **MongoDB Atlas Vector Store** | Vector Database | Vectors in MongoDB Atlas |
| **PGVector Vector Store** | Vector Database | PostgreSQL vector storage |
| **Pinecone Vector Store** | Vector Database | Managed production vector DB |
| **Qdrant Vector Store** | Vector Database | Vector similarity search |
| **Redis Vector Store** | Vector Database | Fast in-memory embeddings |
| **Simple Vector Store** | Vector Database | In-memory (testing only) |
| **Supabase Vector Store** | Vector Database | Vector storage in Supabase |
| **Weaviate Vector Store** | Vector Database | Semantic hybrid search |
| **Zep Vector Store** | Vector Database | Memory + vector storage |

---

## AI / Sub-nodes (63 nodes)

### Chat Model Sub-nodes (attach to any Root node)

| Node | Provider | ⚠️ Notes |
|------|---------|----------|
| Anthropic Chat Model | Anthropic (Claude) | Token limits |
| AWS Bedrock Chat Model | AWS | AWS config required ⚠️ |
| Azure OpenAI Chat Model | Azure | Setup required |
| Cohere Chat Model | Cohere | API limits |
| DeepSeek Chat Model | DeepSeek | |
| Google Gemini Chat Model | Google | API limits |
| Google Vertex Chat Model | Google Vertex | Complex setup |
| Groq Chat Model | Groq | Limited models, very fast |
| Lemonade Chat Model | Lemonade | Paid |
| Mistral Cloud Chat Model | Mistral AI | API limits |
| Ollama Chat Model | Local (Ollama) | Needs local server ⚠️ |
| OpenAI Chat Model | OpenAI | Cost per token ⚠️ |
| OpenRouter Chat Model | OpenRouter | Model variability |
| Vercel AI Gateway Chat Model | Vercel | API limits |
| xAI Grok Chat Model | xAI | Limited access ⚠️ |

### Embeddings Sub-nodes

| Node | Provider | ⚠️ Notes |
|------|---------|----------|
| Embeddings AWS Bedrock | AWS | AWS setup ⚠️ |
| Embeddings Azure OpenAI | Azure | Azure config ⚠️ |
| Embeddings Cohere | Cohere | Paid API |
| Embeddings Google Gemini | Google | API limits |
| Embeddings Google PaLM | Google | Deprecated soon ⚠️ |
| Embeddings Google Vertex | Google Vertex | Setup needed |
| Embeddings HuggingFace Inference | HuggingFace | Rate limits |
| Embeddings Lemonade | Lemonade | Paid |
| Embeddings Mistral Cloud | Mistral | API limits |
| Embeddings Ollama | Local (Ollama) | Needs local server ⚠️ |
| Embeddings OpenAI | OpenAI | Cost ⚠️ |

### Memory Sub-nodes

| Node | Storage | Persistence | ⚠️ Notes |
|------|---------|-------------|----------|
| Chat Memory Manager | Configurable | Configurable | |
| MongoDB Chat Memory | MongoDB | Persistent | DB setup |
| Motorhead | External service | Persistent | |
| Postgres Chat Memory | PostgreSQL | Persistent | SQL setup |
| Redis Chat Memory | Redis | Volatile | Can lose data ⚠️ |
| Simple Memory | In-memory | Session only | Not persistent ⚠️ |
| Xata | Cloud DB | Persistent | API limits |
| Zep | External service | Long-term | External infra |

### Tool Sub-nodes (for AI Agent)

| Node | What the Agent Can Do | ⚠️ Notes |
|------|-----------------------|----------|
| AI Agent Tool | Nest another AI agent as a tool | Needs agent |
| Calculator | Perform math operations | Limited scope |
| Call n8n Workflow Tool | Invoke another n8n workflow | Requires workflow |
| Custom Code Tool | Run custom JS/Python logic | Risk ⚠️ |
| MCP Client Tool | Connect MCP services | Setup required |
| SearXNG Tool | Self-hosted search | Self-hosted ⚠️ |
| SerpApi (Google Search) | Perform Google searches | Paid API |
| Think Tool | Enable chain-of-thought reasoning | Experimental ⚠️ |
| Vector Store Question Answer Tool | Q&A over vectors | Needs setup |
| Wikipedia | Fetch Wikipedia content | Public data |
| Wolfram Alpha | Computation engine | Paid |

### Other Sub-nodes

| Node | Category | Purpose | ⚠️ Notes |
|------|----------|---------|----------|
| Auto-fixing Output Parser | Output Parser | Fix LLM formatting errors | Not perfect ⚠️ |
| Item List Output Parser | Output Parser | Parse lists from LLM | Needs format spec |
| Structured Output Parser | Output Parser | Enforce schema output | Errors if mismatch ⚠️ |
| Contextual Compression Retriever | Retriever | Compress retrieved data | Complex |
| MultiQuery Retriever | Retriever | Multiple query search | Costs more ⚠️ |
| Vector Store Retriever | Retriever | Retrieve from vector DB | Needs embeddings |
| Workflow Retriever | Retriever | Retrieve workflow data | Limited use |
| Character Text Splitter | Text Splitter | Split text by characters | May cut context |
| Recursive Character Text Splitter | Text Splitter | Smart text splitting | Better results |
| Token Splitter | Text Splitter | Split by token count | Needs tokenizer |
| Default Data Loader | Data Loader | Load generic data | |
| GitHub Document Loader | Data Loader | Load files from GitHub | Rate limits |
| Reranker Cohere | Reranker | Improve search result quality | Cost ⚠️ |
| Model Selector | Utility | Choose best model dynamically | Needs tuning |

---

## Related Documents

| Document | Description |
|----------|-------------|
| [`01-core-and-trigger-nodes.md`](./01-core-and-trigger-nodes.md) | Core and trigger node details with parameters and code examples |
| [`02-webhook-deep-dive.md`](./02-webhook-deep-dive.md) | Complete Webhook node guide |
| [`03-ai-nodes-and-open-source-models.md`](./03-ai-nodes-and-open-source-models.md) | AI cluster nodes, local LLM setup (Ollama), NVIDIA NIM, all open-source APIs |
| [`04-community-and-custom-nodes.md`](./04-community-and-custom-nodes.md) | Community nodes and custom node development |

# n8n Resources — Complete Reference Library

> **Author:** Shrinivas Ramaprasad | **Last Updated:** May 2026
>
> A structured, field-by-field reference for everything n8n — hosting, Docker, ngrok, nodes, credentials, integrations, AI/LLM, workflow templates, and automation use cases.

---

## Table of Contents

### 🚀 Getting Started
- [What is n8n?](#what-is-n8n)
- [Quick Decision: Which hosting method?](#quick-decision-which-hosting-method)
- [Quick Decision: Which trigger?](#quick-decision-which-trigger)
- [Quick Decision: Which messaging app?](#quick-decision-which-messaging-app)

### 🖥️ Hosting (How to Run n8n)
- [01 · Local Install (Windows/Mac/Linux without Docker)](#01--local-install)
- [02 · Docker — Recommended Method](#02--docker)
- [03 · Docker + ngrok — Expose to Internet](#03--docker--ngrok)
- [04 · Triggers with ngrok (Telegram/Slack/Discord/WhatsApp)](#04--triggers-with-ngrok)
- [05 · VPS & Cloud Hosting (Hostinger, DigitalOcean, Hetzner)](#05--vps--cloud-hosting)
- [06 · Database Setup (SQLite vs PostgreSQL)](#06--database-setup)
- [07 · Hosting Challenges & Solutions](#07--hosting-challenges)
- [08 · Connect n8n with Claude (MCP/API)](#08--connect-n8n-with-claude)

### 🔑 Credentials
- [All Credential Types — How to Get and Configure](#credentials)
  - [API Keys (OpenAI, Anthropic, Groq, NVIDIA)](#api-keys)
  - [Google OAuth2 & Service Account](#google-oauth2)
  - [Telegram, Slack, GitHub, SMTP](#messaging--dev-tools)
  - [MySQL & PostgreSQL](#database-credentials)

### ⚙️ Nodes Reference
- [Node Categories Overview](#node-categories)
- [Trigger Nodes — All 8+ Types, Step-by-Step](#trigger-nodes)
- [Webhook Node — Complete Deep Dive](#webhook-node)
- [Core Nodes — Every Field with Samples](#core-nodes)
- [AI Nodes — Root Nodes & Sub-nodes](#ai-nodes)
- [Community & Custom Nodes](#community--custom-nodes)
- [Keyboard Shortcuts](#keyboard-shortcuts)

### 🔗 Integrations
- [Google Products (Sheets, Gmail, Drive, Calendar, Docs)](#google-products)
- [Telegram — Complete Guide](#telegram)
- [SQL Databases (MySQL, PostgreSQL, joins, automation)](#sql-databases)
- [GitHub Integration (files, backups, tracker)](#github-integration)

### 🤖 AI / LLM
- [Open-Source Model APIs (Groq, OpenRouter, NVIDIA, HuggingFace)](#open-source-model-apis)
- [Local LLM Setup (Ollama, LM Studio, GPT4All)](#local-llm-setup)
- [CPU-Only LLMs — No GPU Required](#cpu-only-llms-no-gpu-required)
- [Model Rate Limits — All Free Providers](#model-rate-limits)

### 📋 Workflow Templates & Use Cases
- [Save n8n Workflows to GitHub (Auto-Backup)](#save-workflows-to-github)
- [IT Automation Templates](#it-automation-templates)
- [CareerForge Automation Templates](#careerforge-automation-templates)
- [Data Sync & Reporting Templates](#data-sync--reporting-templates)

### ✅ Best Practices
- [Security, Performance, Credential Management](#best-practices)

---

## What is n8n?

n8n is an open-source, self-hostable workflow automation platform. Think Zapier or Make.com — but you own it, host it yourself, and it's free for self-hosting.

```
External Events          n8n Workflow              Outputs
───────────────         ──────────────            ──────────
Telegram message  ──▶   Trigger Node        ──▶   Send reply
Stripe payment    ──▶   ──▶ Core Nodes      ──▶   Update CRM
GitHub push       ──▶   ──▶ Code Nodes      ──▶   Notify Slack
Form submission   ──▶   ──▶ AI Nodes        ──▶   Save to DB
Schedule/cron     ──▶   ──▶ Integration     ──▶   Export report
                              Nodes
```

**Official docs:** https://docs.n8n.io  
**n8n Workflow Templates:** https://n8n.io/workflows/  
**n8n Community:** https://community.n8n.io

---

## Quick Decision: Which Hosting Method?

| Your Situation | Use |
|---|---|
| Just testing / learning | Local npm install OR Docker (laptop) |
| Want Telegram/GitHub webhooks to work | Docker + ngrok |
| Need 24/7 uptime | VPS (Hostinger/Hetzner/DigitalOcean) |
| Already on a NAS (QNAP/Synology) | Docker on NAS |
| Team use, always-on, production | VPS + PostgreSQL + SSL |

---

## Quick Decision: Which Trigger?

| Goal | Trigger Node |
|---|---|
| Run manually / test | Manual Trigger |
| Run daily at 9 AM | Schedule Trigger |
| React to incoming HTTP | Webhook |
| React to Telegram message | Telegram Trigger (polling — no ngrok needed) |
| React to GitHub push/PR | GitHub Trigger |
| React to Gmail | Gmail Trigger (polling) |
| Build a chatbot | Chat Trigger + AI Agent |
| Collect form data | n8n Form Trigger |
| Sub-workflow / function | Execute Sub-workflow Trigger |

---

## Quick Decision: Which Messaging App?

| | Telegram 🥇 | Slack 🥈 | Discord 🥉 | WhatsApp ⚠️ |
|---|---|---|---|---|
| Setup time | 5 min | 20 min | 15 min | 1–3 days |
| Cost | Free | Freemium | Free | Paid after 1K msg |
| Works without ngrok | ✅ Polling mode | ❌ | ❌ | ❌ |
| n8n native node | ✅ | ✅ | ✅ | Via HTTP Request |
| Send & Wait built-in | ✅ | ❌ | ❌ | ❌ |
| Best for | Personal/team bots | Corporate teams | Dev communities | Customer-facing only |

**Recommendation: Start with Telegram. No ngrok needed for basic use.**

---

## Hosting

### 01 · Local Install

**File:** [`hosting/01-host-n8n-local.md`](./hosting/01-host-n8n-local.md)

| Platform | Method | What it covers |
|---|---|---|
| Windows | npm install | Node.js setup, PM2 background service, env vars |
| macOS | Homebrew + npm | Homebrew, permissions fix, PM2 auto-start |
| Linux | npm + systemd | systemd service file, every config field |

**Key env vars to always set:**
```
N8N_ENCRYPTION_KEY=  (32-char hex — generate once, never change)
GENERIC_TIMEZONE=Asia/Kolkata
WEBHOOK_URL=https://your-domain.com
```

---

### 02 · Docker

**File:** [`hosting/02-host-n8n-docker.md`](./hosting/02-host-n8n-docker.md)  
**Expanded guide:** [`hosting/03-docker-ngrok-guide.md`](./hosting/03-docker-ngrok-guide.md)

**Why Docker over direct install:**

| Direct npm | Docker |
|---|---|
| Breaks when Node.js updates | Isolated — never breaks |
| Hard to move to another machine | Copy 2 files = done |
| No auto-restart on crash | `restart: unless-stopped` |
| Hard to add PostgreSQL/Ollama | One docker-compose.yml |

**Minimum working docker-compose.yml:**
```yaml
version: '3.8'
services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    env_file:
      - .env
    volumes:
      - n8n_data:/home/node/.n8n
      # ⚠️ CRITICAL: without this, ALL workflows are lost on restart
volumes:
  n8n_data:
```

**Official Docker install docs:** https://docs.n8n.io/hosting/installation/docker/

---

### 03 · Docker + ngrok

**File:** [`hosting/03-docker-ngrok-guide.md`](./hosting/03-docker-ngrok-guide.md)

**Why ngrok:** Your local n8n at `localhost:5678` is invisible to the internet. External services (Telegram, Stripe, GitHub) need a public HTTPS URL to send webhooks.

```
Telegram Bot API
      ↓
https://abc123.ngrok-free.app  ← Public URL
      ↓  (encrypted tunnel)
ngrok agent (on your laptop)
      ↓
http://localhost:5678  ← Your local n8n
```

**Quick setup:**
```bash
# 1. Start n8n
docker compose up -d

# 2. Start ngrok (new terminal)
ngrok http 5678

# 3. Copy the https URL shown, e.g.: https://abc123.ngrok-free.app
# 4. Update .env:
WEBHOOK_URL=https://abc123.ngrok-free.app
N8N_EDITOR_BASE_URL=https://abc123.ngrok-free.app

# 5. Restart n8n
docker compose restart n8n

# 6. Open n8n at the ngrok URL (not localhost)
```

**Free alternatives with permanent URL:**
- Cloudflare Tunnel: `cloudflared tunnel --url http://localhost:5678` (free, permanent)
- ngrok paid ($8/mo): fixed domain like `n8n.ngrok.app`

**ngrok inspector:** http://127.0.0.1:4040 — see all incoming requests in real time

---

### 04 · Triggers with ngrok

**File:** [`hosting/04-triggers-with-ngrok.md`](./hosting/04-triggers-with-ngrok.md)

Covers full setup for: Telegram (polling + webhook), Slack (Event API + challenge handling), Discord (webhook + bot), WhatsApp (Meta API + Waha self-hosted).

**Architecture:**
```
Telegram/Slack/GitHub/Stripe
          ↓
  ngrok public URL
          ↓
  Docker n8n container
          ↓
  Trigger node fires workflow
```

---

### 05 · VPS & Cloud Hosting

**File:** [`hosting/05-vps-cloud.md`](./hosting/05-vps-cloud.md)

| Provider | Plan | Cost | Notes |
|---|---|---|---|
| **Hostinger** | KVM 2 (2 vCPU, 8 GB) | ~$8/mo | Good value, easy UI |
| **Hetzner** | CX22 (2 vCPU, 4 GB) | ~€4/mo | Best EU value |
| **DigitalOcean** | Basic 2 GB | $12/mo | Good docs, easy |
| **AWS EC2** | t3.small | ~$15/mo | Use if in AWS ecosystem |
| **Contabo** | VPS S | ~€5/mo | Cheap, good storage |

**With n8n + Ollama:**

| Provider | Recommended Plan | Cost |
|---|---|---|
| Hostinger | KVM 4 (4 vCPU, 16 GB) | ~$15/mo |
| Hetzner | CX32 (4 vCPU, 8 GB) | ~€9/mo |
| Any GPU VPS | NVIDIA T4/A10 | $50–200/mo |

---

### 06 · Database Setup

**File:** [`hosting/database-guide.md`](./hosting/database-guide.md)

| DB | For n8n backend? | For workflows? | When to use |
|---|---|---|---|
| SQLite | ✅ Default | Via code | Testing, personal, < 50 exec/day |
| PostgreSQL | ✅ Recommended | ✅ Built-in node | Production, teams, high volume |
| MySQL | ❌ Deprecated (n8n v1+) | ✅ Built-in node | Workflow data only |
| MongoDB | ❌ | ✅ Built-in node | Document data |
| Redis | ❌ | ✅ Built-in node | Caching, queue mode |

---

### 07 · Hosting Challenges

**File:** [`hosting/03-hosting-challenges.md`](./hosting/03-hosting-challenges.md)

Common challenges: Ollama connection from Docker, credentials lost after move, webhook URL showing localhost, timezone wrong on schedules, SSL errors, OAuth redirect issues.

---

### 08 · Connect n8n with Claude

**File:** [`hosting/06-connect-claude.md`](./hosting/06-connect-claude.md)

How to: Use Claude as an LLM in n8n AI Agent, expose n8n as an MCP server for Claude Desktop, integrate Claude Desktop with local n8n workflows.

---

## Credentials

**Files:**
- [`credentials/01-credentials-part1.md`](./credentials/01-credentials-part1.md) — API keys, Google OAuth2, Service Account
- [`credentials/02-credentials-part2.md`](./credentials/02-credentials-part2.md) — Telegram, Slack, GitHub, SMTP, MySQL, PostgreSQL

### API Keys
OpenAI (`sk-proj-`), Anthropic (`sk-ant-api03-`), Groq (`gsk_`), NVIDIA NIM (`nvapi-`), HuggingFace (`hf_`)

### Google OAuth2
Two options: Quick Sign-In (personal, 2 clicks) or Cloud Console (Workspace, custom app). Redirect URI must be exact: `https://your-n8n.com/rest/oauth2-credential/callback`

### Messaging & Dev Tools
- Telegram: BotFather → `/newbot` → copy token → get Chat ID via `getUpdates`
- Slack: Create app → add scopes → install → copy `xoxb-` token → `/invite @bot` to channels
- GitHub: Fine-grained PAT → select repos → Contents: Read+Write

### Database Credentials
- MySQL: `Host | Database | User | Password | Port(3306) | SSL`
- PostgreSQL: `Host | Database | User | Password | Port(5432) | SSL | Schema(public)`

**Master rule:** `N8N_ENCRYPTION_KEY` must never change after first run — losing it = losing all credentials.

---

## Nodes Reference

### Node Categories

**File:** [`nodes/00-node-categories.md`](./nodes/00-node-categories.md)

| Category | Purpose | Can start workflow? |
|---|---|---|
| Trigger | Start on event | ✅ Yes |
| Core/Regular | Transform, route, filter | ❌ No |
| Code | Custom JS/Python | ❌ No |
| Webhook | HTTP receive/respond | Trigger can |
| HTTP | Outbound API calls | ❌ No |
| AI | LLM pipelines | ❌ No |
| App/Integration | External services | App triggers can |

### Trigger Nodes

**File:** [`nodes/01-trigger-nodes-complete.md`](./nodes/01-trigger-nodes-complete.md)

All 8 trigger types with step-by-step implementation: Manual, Schedule (cron), Webhook, App Events (Gmail/Slack/GitHub), Form, Sub-workflow, Chat, Error, Activation.

### Webhook Node

**File:** [`nodes/02-webhook-deep-dive.md`](./nodes/02-webhook-deep-dive.md)

Every parameter: HTTP Method (GET/POST/PUT/PATCH/DELETE), Path, Authentication (None/Basic/Header Auth/JWT), Response Mode (Immediately/Last Node/Respond Node), Response Code, Raw Body.

Testing methods: n8n test mode, curl, **Postman step-by-step**, ngrok inspector, webhook.site.

Service configs: Stripe, GitHub, Slack, HubSpot, Shopify, **Postman**.

### Core Nodes

**File:** [`nodes/05-node-fields-guide.md`](./nodes/05-node-fields-guide.md)

Every field with samples for: Edit Fields (Set), Filter, If, Switch, Merge, Loop Over Items, Wait, Code (JS+Python), HTTP Request, Schedule Trigger, Google Sheets, Gmail, Slack, Telegram, AI Agent.

### AI Nodes

**File:** [`nodes/03-ai-nodes-and-open-source-models.md`](./nodes/03-ai-nodes-and-open-source-models.md)

Root nodes: AI Agent, Basic LLM Chain, Q&A Chain, Summarization, Information Extractor, Text Classifier.  
Sub-nodes: 28 Chat Models, Memory types, Tools, Vector Stores, Embeddings, Text Splitters.

### Community & Custom Nodes

**File:** [`nodes/04-community-and-custom-nodes.md`](./nodes/04-community-and-custom-nodes.md)

Install community nodes (3 methods), build TypeScript custom nodes, publish to npm.

### Keyboard Shortcuts

**File:** [`keyboard-shortcuts/n8n-keyboard-shortcuts.md`](./keyboard-shortcuts/n8n-keyboard-shortcuts.md)

All shortcuts for: Workflow controls, Canvas navigation, Node operations, Node panel, Code editor, Command bar (Ctrl+K). Win/Mac variants.

---

## Integrations

### Google Products

**File:** [`integrations/02-google-products.md`](./integrations/02-google-products.md)

| Node | Key Operations |
|---|---|
| Google Sheets | Read rows, append row, update row, append-or-update, clear |
| Gmail | Send email (HTML), read/filter emails, download attachments |
| Google Drive | Upload, download (as PDF/XLSX), search, move files |
| Google Calendar | Create events, add Google Meet, invite attendees |
| Google Docs | Create from template, find & replace placeholders |
| Google Translate | Translate text (auto-detect source) |
| Google Analytics | Get reports, dimensions, metrics |
| YouTube | Upload videos, manage playlists |

### Telegram

**File:** [`integrations/03-telegram-and-messaging.md`](./integrations/03-telegram-and-messaging.md)

All operations: Send Message (HTML/Markdown), Send Photo, Send Document, Send and Wait for Response (Approval/Free Text/Custom Form), Telegram Trigger.

Complete workflow example: `/ticket` command → Google Sheets log → Slack notify → Telegram reply.

### SQL Databases

**File:** [`integrations/04-local-sql-database.md`](./integrations/04-local-sql-database.md)

MySQL + PostgreSQL: parameterised queries, upsert patterns, cross-table JOINs, scheduled sync, archive old records, health monitoring. Docker networking for all scenarios.

### GitHub Integration

**File:** [`integrations/01-github-integration.md`](./integrations/01-github-integration.md)

7 use cases: Read/write files, tracker with deduplication, scheduled workflow backup, GitHub as config store, trigger n8n from GitHub events, save reports as Markdown.

---

## AI / LLM

### Open-Source Model APIs

**File:** [`nodes/03-ai-nodes-and-open-source-models.md`](./nodes/03-ai-nodes-and-open-source-models.md)

| Provider | Free Tier | n8n Node | Best For |
|---|---|---|---|
| Ollama | ✅ Free (local) | Ollama Chat Model | Privacy, offline, zero cost |
| Groq | ✅ 14K req/day | Groq Chat Model | Fastest free cloud inference |
| OpenRouter | ✅ Free models | OpenRouter Chat Model | 100+ models, one API key |
| NVIDIA NIM | ✅ Free credits | HTTP Request / community node | GPU-optimised models |
| Hugging Face | ✅ Rate-limited | HF Inference Model | Research, 100K+ models |
| Mistral | ✅ Experiment tier | Mistral Chat Model | EU data residency |
| Cohere | ✅ 1K req/month | Cohere Chat Model | RAG + reranking |
| DeepSeek | Low cost | DeepSeek Chat Model | Strong reasoning |
| Google Gemini | ✅ Free tier | Gemini Chat Model | Long context, multimodal |

### Local LLM Setup

**File:** [`nodes/03-ai-nodes-and-open-source-models.md`](./nodes/03-ai-nodes-and-open-source-models.md) — Part 4

Step-by-step for Ollama (primary), LM Studio, GPT4All. 4 Docker scenarios: same machine, Docker on Windows/Mac, Docker on Linux, n8n Cloud + ngrok.

### CPU-Only LLMs — No GPU Required

**File:** [`ai-llm/01-cpu-only-llms.md`](./ai-llm/01-cpu-only-llms.md)

Models that run on CPU (no GPU) with RAM requirements, tokens/second benchmarks, n8n workflow configuration, and fallback strategies when context limits are hit.

| Model | Min RAM | Tokens/sec (CPU) | Best For |
|---|---|---|---|
| Qwen3:0.6B | 4 GB | 15–25 t/s | Ultra-light, fast responses |
| Gemma3:1B | 4 GB | 10–20 t/s | Light chatbot |
| Phi-3 Mini (3.8B Q4) | 6 GB | 8–15 t/s | Best quality/RAM balance |
| Llama3.2:3B (Q4) | 6 GB | 8–12 t/s | General purpose |
| Llama3.1:8B (Q4) | 10 GB | 5–10 t/s | Good quality, needs more RAM |
| Mistral:7B (Q4) | 10 GB | 4–8 t/s | Long context |

### Model Rate Limits

**File:** [`nodes/06-open-source-model-limits.md`](./nodes/06-open-source-model-limits.md)

Daily/hourly/per-minute limits for every free provider, plus strategies for handling limits in n8n (Wait node, Retry On Fail, multi-provider fallback chains).

---

## Workflow Templates & Use Cases

### Save Workflows to GitHub

**File:** [`workflows/01-save-workflows-to-github.md`](./workflows/01-save-workflows-to-github.md)

Yes — n8n can automatically save all workflows to a GitHub repo. Two methods:
1. **Built-in Source Control** (Enterprise): Git integration in Settings
2. **Workflow-based backup** (Free): n8n API → GitHub node (covered in detail)

Official templates: https://n8n.io/workflows/1534 | https://n8n.io/workflows/2532 | https://n8n.io/workflows/4064

### IT Automation Templates

**File:** [`workflows/02-use-case-templates.md`](./workflows/02-use-case-templates.md)

Ready-to-use workflow JSON for: IT ticket bot (Telegram), daily report, Gmail-to-ticket, approval flow, scheduled DB sync.

### CareerForge Automation Templates

**File:** [`workflows/02-use-case-templates.md`](./workflows/02-use-case-templates.md)

Automation for: LinkedIn recruiter outreach (SPARK), CV tailor chain, lead qualifier, CareerForge content pipeline.

### Data Sync & Reporting Templates

**File:** [`workflows/02-use-case-templates.md`](./workflows/02-use-case-templates.md)

Workflows for: CRM sync, Google Sheets ETL, scheduled email report, deduplication tracker.

---

## Best Practices

**File:** [`best-practices/01-best-practices.md`](./best-practices/01-best-practices.md)

Covers: Workflow naming, error handling, n8n Variables, credential management, webhook security, MCP integration, app mapping, performance (batching, Wait nodes, pruning).

**One-page checklist:**
```
Workflow Design:
  ✅ Name nodes descriptively (F2 to rename)
  ✅ Add notes to complex nodes (Settings tab)
  ✅ Handle errors (Error Trigger + Slack/Telegram alert)
  ✅ Set GENERIC_TIMEZONE explicitly
  ✅ Test with 1 item → 10 items → activate

Webhooks:
  ✅ Always use Header Auth
  ✅ Use ngrok URL in WEBHOOK_URL env var
  ✅ Use Respond: Immediately for external services

AI Workflows:
  ✅ System prompt includes today's date
  ✅ Set Max Iterations on AI Agent
  ✅ Use Redis/Postgres memory for production

Hosting:
  ✅ HTTPS everywhere
  ✅ Back up N8N_ENCRYPTION_KEY (password manager)
  ✅ Enable execution pruning
  ✅ PostgreSQL for production (not SQLite)
```

---

*Maintained by [Shrinivas Ramaprasad](https://github.com/Shri-Phnx)*  
*Repository: https://github.com/Shri-Phnx/n8n-resources*

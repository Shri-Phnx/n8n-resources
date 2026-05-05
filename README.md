# n8n Resources — Complete Reference Library

> **Author:** Shrinivas Ramaprasad | **Updated:** May 2026
> Field-by-field n8n reference — hosting, Docker, ngrok, nodes, credentials, integrations, AI/LLM, and automation templates.

---

## Table of Contents

### 🚀 Getting Started
- [What is n8n?](#what-is-n8n)
- [Quick Decision: Hosting](#quick-decision-hosting)
- [Quick Decision: Trigger](#quick-decision-trigger)
- [Quick Decision: Messaging App](#quick-decision-messaging-app)

### 🖥️ Hosting
- [01 · Local Install (Windows / Mac / Linux)](#01--local-install)
- [02 · Docker — Recommended](#02--docker)
- [03 · Docker + ngrok — Expose to Internet](#03--docker--ngrok)
- [04 · Telegram / Slack / Discord / WhatsApp + ngrok](#04--messaging-triggers-with-ngrok)
- [05 · VPS & Cloud (Hostinger, Hetzner, DigitalOcean)](#05--vps--cloud)
- [06 · Database (SQLite vs PostgreSQL)](#06--database)
- [07 · Hosting Challenges & Fixes](#07--hosting-challenges)
- [08 · Connect n8n with Claude](#08--connect-n8n-with-claude)

### 🔑 Credentials
- [API Keys (OpenAI, Anthropic, Groq, NVIDIA)](#api-keys)
- [Google OAuth2 & Service Account](#google-oauth2)
- [Telegram, Slack, GitHub, SMTP](#telegram-slack-github-smtp)
- [MySQL & PostgreSQL](#mysql--postgresql)

### ⚙️ Nodes
- [Node Categories](#node-categories)
- [Trigger Nodes — All Types, Step-by-Step](#trigger-nodes)
- [Webhook — Every Parameter](#webhook)
- [Core Nodes — Every Field with Samples](#core-nodes)
- [AI Nodes — Root + Sub-nodes](#ai-nodes)
- [Community & Custom Nodes](#community--custom-nodes)
- [Keyboard Shortcuts](#keyboard-shortcuts)

### 🔗 Integrations
- [Google Products (Sheets, Gmail, Drive, Calendar, Docs)](#google-products)
- [Telegram — Complete Guide](#telegram)
- [SQL Databases (MySQL, PostgreSQL, joins)](#sql-databases)
- [GitHub — Files, Backups, Tracker](#github)

### 🤖 AI / LLM
- [Open-Source APIs (Groq, OpenRouter, NVIDIA, HuggingFace)](#open-source-apis)
- [Local LLM — Ollama Setup](#local-llm-setup)
- [CPU-Only LLMs — No GPU Required](#cpu-only-llms)
- [Model Rate Limits — All Free Providers](#model-rate-limits)

### 📋 Workflows & Use Cases
- [Save n8n Workflows to GitHub](#save-workflows-to-github)
- [Pure Automation Workflows (No AI Required)](#pure-automation-workflows)
- [Importable JSON Templates](#importable-json-templates)
- [Use Case Templates (IT, CareerForge, Reporting)](#use-case-templates)

### ✅ Best Practices
- [Security, Performance, Checklists](#best-practices)

---

## What is n8n?

n8n is open-source, self-hostable workflow automation. Like Zapier or Make.com — but you own it, host it, and it is free.

```
Telegram message  →  Trigger Node  →  Core Nodes  →  Send reply
Stripe payment    →  Trigger Node  →  Code Nodes  →  Update CRM
GitHub push       →  Trigger Node  →  AI Nodes    →  Notify Slack
Schedule/cron     →  Trigger Node  →  Integrations →  Export report
```

**Docs:** https://docs.n8n.io | **Templates:** https://n8n.io/workflows/ | **Community:** https://community.n8n.io

---

## Quick Decision: Hosting

| Situation | Use |
|---|---|
| Testing / learning | Docker on laptop |
| Telegram / GitHub webhooks needed | Docker + ngrok |
| Need 24/7 uptime | VPS (Hostinger / Hetzner) |
| NAS (QNAP / Synology) | Docker on NAS |
| Team / production | VPS + PostgreSQL + SSL |

---

## Quick Decision: Trigger

| Goal | Trigger |
|---|---|
| Test manually | Manual Trigger |
| Run at 9 AM daily | Schedule Trigger |
| Receive HTTP event | Webhook Trigger |
| React to Telegram message | Telegram Trigger (polling — no ngrok needed) |
| React to GitHub push | GitHub Trigger |
| React to Gmail | Gmail Trigger (polling) |
| Build a chatbot | Chat Trigger + AI Agent |
| Collect form data | n8n Form Trigger |

---

## Quick Decision: Messaging App

| | Telegram 🥇 | Slack 🥈 | Discord 🥉 | WhatsApp ⚠️ |
|---|---|---|---|---|
| Setup time | 5 min | 20 min | 15 min | 1–3 days |
| Cost | Free | Freemium | Free | Paid after 1K msg |
| Works without ngrok | ✅ Polling mode | ❌ | ❌ | ❌ |
| n8n native node | ✅ | ✅ | ✅ | Via HTTP Request |
| Send and Wait built-in | ✅ | ❌ | ❌ | ❌ |

**Start with Telegram — no ngrok needed for basic use.**

---

## Hosting

### 01 · Local Install

**File:** [`hosting/01-host-n8n-local.md`](./hosting/01-host-n8n-local.md)

Covers: Node.js install, PM2 background service, env vars — Windows, macOS, Linux.

Key env vars:
```
N8N_ENCRYPTION_KEY=<32-char hex, generate once, never change>
GENERIC_TIMEZONE=Asia/Kolkata
WEBHOOK_URL=https://your-domain.com
```

### 02 · Docker

**File:** [`hosting/03-docker-ngrok-guide.md`](./hosting/03-docker-ngrok-guide.md)

Minimum `docker-compose.yml`:
```yaml
version: '3.8'
services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    env_file: .env
    volumes:
      - n8n_data:/home/node/.n8n
volumes:
  n8n_data:
```

Ref: https://docs.n8n.io/hosting/installation/docker/

### 03 · Docker + ngrok

**File:** [`hosting/03-docker-ngrok-guide.md`](./hosting/03-docker-ngrok-guide.md)

```
Telegram/Stripe/GitHub  →  https://abc.ngrok-free.app  →  ngrok agent  →  localhost:5678 (n8n)
```

```bash
docker compose up -d        # Start n8n
ngrok http 5678             # New terminal — start tunnel
# Copy the https:// URL, update .env:
WEBHOOK_URL=https://abc123.ngrok-free.app
docker compose restart n8n  # Apply change
```

Free permanent URL alternative: `cloudflared tunnel --url http://localhost:5678`

### 04 · Messaging Triggers with ngrok

**File:** [`hosting/04-triggers-with-ngrok.md`](./hosting/04-triggers-with-ngrok.md)

Full setup: Telegram (polling + webhook), Slack (Event API), Discord (bot + webhook), WhatsApp (Meta API + Waha self-hosted).

### 05 · VPS & Cloud

**File:** [`hosting/05-vps-cloud.md`](./hosting/05-vps-cloud.md)

| Provider | Plan | Cost | Notes |
|---|---|---|---|
| Hostinger | KVM 2 (2 vCPU, 8 GB) | ~$8/mo | Best value for beginners |
| Hetzner | CX22 (2 vCPU, 4 GB) | ~€4/mo | Best EU value |
| DigitalOcean | Basic 2 GB | $12/mo | Great docs |
| Contabo | VPS S | ~€5/mo | Most storage |

### 06 · Database

**File:** [`hosting/database-guide.md`](./hosting/database-guide.md)

| DB | n8n backend? | Workflow target? | Use when |
|---|---|---|---|
| SQLite | ✅ Default | Via code | Testing, < 50 exec/day |
| PostgreSQL | ✅ Production | ✅ Built-in | Production, teams |
| MySQL | ❌ Deprecated v1+ | ✅ Built-in | Workflow data only |
| MongoDB | ❌ | ✅ Built-in | Document data |
| Redis | ❌ | ✅ Built-in | Cache, queue mode |

### 07 · Hosting Challenges

**File:** [`hosting/03-hosting-challenges.md`](./hosting/03-hosting-challenges.md)

Covers: Ollama from Docker, credential loss on move, wrong webhook URL, timezone issues, SSL errors.

### 08 · Connect n8n with Claude

**File:** [`hosting/06-connect-claude.md`](./hosting/06-connect-claude.md)

Three methods: Claude API as LLM in AI Agent, n8n as MCP server for Claude Desktop, simple webhook.

---

## Credentials

**Files:**
- [`credentials/01-credentials-part1.md`](./credentials/01-credentials-part1.md) — API keys + Google OAuth2 + Service Account
- [`credentials/02-credentials-part2.md`](./credentials/02-credentials-part2.md) — Telegram, Slack, GitHub, SMTP, MySQL, PostgreSQL

### API Keys

| Service | Format | Get from |
|---|---|---|
| OpenAI | `sk-proj-...` | platform.openai.com → API keys |
| Anthropic | `sk-ant-api03-...` | console.anthropic.com → API Keys |
| Groq | `gsk_...` | console.groq.com → API Keys |
| NVIDIA NIM | `nvapi-...` | build.nvidia.com → API Keys |

### Google OAuth2

Option A: Quick Sign-In (personal, 2 clicks).
Option B: Cloud Console (Workspace). Redirect URI must be exact:
`https://your-n8n.com/rest/oauth2-credential/callback`

### Telegram, Slack, GitHub, SMTP

- **Telegram:** BotFather → `/newbot` → copy token → get Chat ID via `getUpdates`
- **Slack:** Create app → add scopes → install → copy `xoxb-` token → `/invite @bot`
- **GitHub:** Fine-grained PAT → select repos → Contents: Read+Write
- **Gmail SMTP:** myaccount.google.com → App passwords → 16-char password

### MySQL & PostgreSQL

- MySQL: `Host | Database | User | Password | Port 3306 | SSL`
- PostgreSQL: `Host | Database | User | Password | Port 5432 | SSL | Schema: public`

> **Critical:** `N8N_ENCRYPTION_KEY` must never change after first run.

---

## Nodes

### Node Categories

**File:** [`nodes/00-node-categories.md`](./nodes/00-node-categories.md)

| Category | Purpose | Starts workflow? |
|---|---|---|
| Trigger | Start on event | ✅ Yes |
| Core / Regular | Transform, route, filter | ❌ No |
| Code | Custom JS or Python | ❌ No |
| Webhook | HTTP receive / respond | Trigger can |
| HTTP | Outbound API calls | ❌ No |
| AI | LLM pipelines | ❌ No |
| App / Integration | External service actions | App triggers can |

### Trigger Nodes

**File:** [`nodes/01-trigger-nodes-complete.md`](./nodes/01-trigger-nodes-complete.md)

All 8+ trigger types: Manual, Schedule (cron), Webhook, App Events (Gmail/Slack/GitHub), Form, Sub-workflow, Chat, Error, Activation.

### Webhook

**File:** [`nodes/02-webhook-deep-dive.md`](./nodes/02-webhook-deep-dive.md)

Every parameter: Method, Path, Auth (None/Basic/Header/JWT), Response Mode, Raw Body.
Testing: n8n test mode, curl, Postman (step-by-step), ngrok inspector, webhook.site.
Service configs: Stripe, GitHub, Slack, HubSpot, Shopify, Postman.

### Core Nodes

**File:** [`nodes/05-node-fields-guide.md`](./nodes/05-node-fields-guide.md)

Every field with samples: Edit Fields, Filter, If, Switch, Merge, Loop, Wait, Code (JS+Python), HTTP Request, Schedule Trigger, Google Sheets, Gmail, Slack, Telegram, AI Agent.

### AI Nodes

**File:** [`nodes/03-ai-nodes-and-open-source-models.md`](./nodes/03-ai-nodes-and-open-source-models.md)

Root: AI Agent, Basic LLM Chain, Q&A Chain, Summarization, Info Extractor, Text Classifier.
Sub-nodes: 28 Chat Models, Memory, Tools, Vector Stores, Embeddings, Splitters.

### Community & Custom Nodes

**File:** [`nodes/04-community-and-custom-nodes.md`](./nodes/04-community-and-custom-nodes.md)

### Keyboard Shortcuts

**File:** [`keyboard-shortcuts/n8n-keyboard-shortcuts.md`](./keyboard-shortcuts/n8n-keyboard-shortcuts.md)

All shortcuts: Workflow controls, Canvas navigation, Node ops, Code editor, Command bar (Ctrl+K).

---

## Integrations

### Google Products

**File:** [`integrations/02-google-products.md`](./integrations/02-google-products.md)

| Node | Key Operations |
|---|---|
| Sheets | Read, append, update, append-or-update |
| Gmail | Send HTML email, read/filter, attachments |
| Drive | Upload, download as PDF/XLSX, search |
| Calendar | Create events, Google Meet, attendees |
| Docs | Create from template, find and replace |
| Translate | Auto-detect source, 50+ languages |

### Telegram

**File:** [`integrations/03-telegram-and-messaging.md`](./integrations/03-telegram-and-messaging.md)

All ops: Send Message, Send Photo, Send Document, Send and Wait (Approval/Free Text/Form).
Full example: `/ticket` command → Sheets log → Slack notify → Telegram reply.

### SQL Databases

**File:** [`integrations/04-local-sql-database.md`](./integrations/04-local-sql-database.md)

MySQL + PostgreSQL: queries, upsert, JOINs, scheduled sync, archive, health monitor, Docker networking.

### GitHub

**File:** [`integrations/01-github-integration.md`](./integrations/01-github-integration.md)

7 use cases: files, tracker deduplication, scheduled backup, config store, CI/CD triggers, reports.

---

## AI / LLM

### Open-Source APIs

**File:** [`nodes/03-ai-nodes-and-open-source-models.md`](./nodes/03-ai-nodes-and-open-source-models.md)

| Provider | Free Tier | Best For |
|---|---|---|
| Ollama | ✅ Free (local) | Privacy, offline, zero cost |
| Groq | ✅ 14K req/day | Fastest free cloud |
| OpenRouter | ✅ Free models | 100+ models, one key |
| NVIDIA NIM | ✅ Free credits | GPU-optimised models |
| Hugging Face | ✅ Rate-limited | 100K+ research models |
| Google Gemini | ✅ Free tier | Long context, multimodal |
| Mistral | ✅ Experiment tier | EU data residency |

### Local LLM Setup

**File:** [`nodes/03-ai-nodes-and-open-source-models.md`](./nodes/03-ai-nodes-and-open-source-models.md) — Part 4

Ollama (primary), LM Studio, GPT4All. Four Docker scenarios covered.

### CPU-Only LLMs

**File:** [`ai-llm/01-cpu-only-llms.md`](./ai-llm/01-cpu-only-llms.md)

| Model | Min RAM | Tokens/sec CPU | Best For |
|---|---|---|---|
| Qwen3:0.6B | 4 GB | 15–25 t/s | Ultra-light tasks |
| Phi-3 Mini Q4 | 6 GB | 8–15 t/s | Best quality/RAM balance |
| Llama3.2:3B Q4 | 6 GB | 8–12 t/s | General purpose |
| Llama3.1:8B Q4 | 10 GB | 5–10 t/s | Good quality |

Includes: multi-provider fallback strategy, context-limit handling (Map Reduce, text trimming).

### Model Rate Limits

**File:** [`nodes/06-open-source-model-limits.md`](./nodes/06-open-source-model-limits.md)

Daily/hourly limits for every free provider, plus Wait node + Retry On Fail strategies.

---

## Workflows & Use Cases

### Save Workflows to GitHub

**File:** [`workflows/01-save-workflows-to-github.md`](./workflows/01-save-workflows-to-github.md)

Two methods: Enterprise Source Control (built-in Git) and free Workflow-based backup (n8n API → GitHub node). Full 12-node workflow with every field explained.

Official templates: https://n8n.io/workflows/1534 · https://n8n.io/workflows/2532 · https://n8n.io/workflows/4064

### Pure Automation Workflows (No AI Required)

**File:** [`workflows/03-pure-automation-workflows.md`](./workflows/03-pure-automation-workflows.md)

Based on Shrinivas's actual use cases — ITAM/ITSM, ServiceNow, CareerForge, job search, reporting. Every workflow uses only triggers, core nodes, and app integrations. No AI agent required.

| Workflow | Trigger | Key Nodes |
|---|---|---|
| IT Telegram Ticket Bot | Telegram | If → Sheets → Slack → Telegram reply |
| Daily ITAM Status Report | Schedule | Sheets → Code → Telegram + Email |
| CareerForge Lead Qualifier | Webhook | Code (score) → Switch → HubSpot + Telegram |
| Recruiter Outreach Tracker | Manual / Schedule | Sheets → Filter → Gmail (SPARK) |
| Job Application Deduplication | Webhook / Form | Sheets upsert → Telegram |
| ServiceNow Incident Monitor | Schedule | HTTP → Filter → Telegram alert |
| n8n Workflow Backup | Schedule | n8n API → Loop → GitHub |
| Gmail to Ticket Tracker | Gmail Trigger | Edit Fields → Sheets → Label |

### Importable JSON Templates

**Folder:** [`workflows/json/`](./workflows/json/)

Ready-to-import `.json` files for n8n. See [`workflows/json/HOW-TO-IMPORT.md`](./workflows/json/HOW-TO-IMPORT.md).

| File | Workflow |
|---|---|
| `01-it-telegram-ticket-bot.json` | Telegram `/ticket` command → Sheets + Slack |
| `02-daily-morning-briefing.json` | 9 AM schedule → Sheets → Telegram report |
| `03-n8n-backup-to-github.json` | Daily 2 AM → n8n API → GitHub commit |
| `04-careerforge-lead-qualifier.json` | Webhook → rule-based score → HubSpot + Telegram |
| `05-gmail-application-tracker.json` | Gmail trigger → Sheets upsert → Telegram notify |

### Use Case Templates (Documented)

**File:** [`workflows/02-use-case-templates.md`](./workflows/02-use-case-templates.md)

Node-by-node instructions for: IT Bot, Daily Report, Gmail CRM, Approval Flow, DB Sync, CareerForge Qualifier.

---

## Best Practices

**File:** [`best-practices/01-best-practices.md`](./best-practices/01-best-practices.md)

```
Workflow Design
  Name nodes descriptively (F2 to rename)
  Add notes to complex nodes (Settings tab)
  Global Error Trigger → Telegram/Slack alert
  Set GENERIC_TIMEZONE explicitly
  Test with 1 item → 10 items → activate

Webhooks
  Always use Header Auth
  Set WEBHOOK_URL in .env to ngrok/domain URL
  Use Respond: Immediately for external services

Hosting
  HTTPS everywhere
  Back up N8N_ENCRYPTION_KEY in password manager
  Enable execution pruning
  PostgreSQL for production (not SQLite)
```

---

## Official References

| Topic | URL |
|---|---|
| n8n Docs | https://docs.n8n.io |
| Docker Install | https://docs.n8n.io/hosting/installation/docker/ |
| All Install Methods | https://docs.n8n.io/hosting/installation/ |
| Environment Variables | https://docs.n8n.io/hosting/configuration/environment-variables/ |
| Webhook Node | https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.webhook/ |
| n8n Community | https://community.n8n.io |
| Workflow Templates | https://n8n.io/workflows/ |

---

*Maintained by [Shrinivas Ramaprasad](https://github.com/Shri-Phnx)*

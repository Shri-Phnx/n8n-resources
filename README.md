# n8n Resources

> **Author:** Shrinivas Ramaprasad
> A curated collection of n8n workflow automation references, cheat sheets, best practices, hosting guides, and integration patterns.
> **Node data source:** N8N_All_Nodes.xlsx — 819 nodes (67 core, 270 actions, 90 triggers, 21 root, 63 sub-nodes, 308 credentials)

---

## Folder Structure

```
n8n-resources/
├── keyboard-shortcuts/    Keyboard shortcuts — all 6 categories, Win + Mac
├── nodes/                 Node references, field guides, AI/LLM, limits
├── hosting/               Hosting guide (all platforms) + database guide
├── integrations/          Step-by-step integration patterns (GitHub, etc.)
└── best-practices/        Workflow design, security, performance
```

---

## Contents

| Folder | File | Description |
|--------|------|-------------|
| [`keyboard-shortcuts/`](./keyboard-shortcuts/) | [`n8n-keyboard-shortcuts.md`](./keyboard-shortcuts/n8n-keyboard-shortcuts.md) | Complete shortcut reference — Windows/Linux & Mac, all 6 categories |
| [`nodes/`](./nodes/) | [`00-nodes-overview.md`](./nodes/00-nodes-overview.md) | Master index of all 819 node types with purpose and use cases |
| [`nodes/`](./nodes/) | [`01-core-and-trigger-nodes.md`](./nodes/01-core-and-trigger-nodes.md) | Core & trigger nodes — detailed parameters, patterns, code examples |
| [`nodes/`](./nodes/) | [`02-webhook-deep-dive.md`](./nodes/02-webhook-deep-dive.md) | Complete Webhook node guide — auth, patterns, security, testing |
| [`nodes/`](./nodes/) | [`03-ai-nodes-and-open-source-models.md`](./nodes/03-ai-nodes-and-open-source-models.md) | AI cluster nodes, all open-source & free API providers, local LLM integration |
| [`nodes/`](./nodes/) | [`04-community-and-custom-nodes.md`](./nodes/04-community-and-custom-nodes.md) | Community node install (3 methods), custom node development end-to-end |
| [`nodes/`](./nodes/) | [`05-node-fields-guide.md`](./nodes/05-node-fields-guide.md) | Field-by-field guide for every major node (Telegram, Webhook, HTTP, Schedule, AI, etc.) |
| [`nodes/`](./nodes/) | [`06-open-source-model-limits.md`](./nodes/06-open-source-model-limits.md) | Daily/rate limits for every free LLM provider — Groq, OpenRouter, NVIDIA, HF, Gemini, Ollama |
| [`hosting/`](./hosting/) | [`complete-hosting-guide.md`](./hosting/complete-hosting-guide.md) | **All-in-one hosting guide** — Windows/Mac/Linux, Docker, Cloud VPS, SSL, every env variable explained |
| [`hosting/`](./hosting/) | [`database-guide.md`](./hosting/database-guide.md) | **Complete database guide** — SQLite vs PostgreSQL, full Postgres setup & maintenance, SQLite→Postgres migration, MongoDB, Redis, Supabase, NocoDB, decision guide |
| [`integrations/`](./integrations/) | [`01-github-integration.md`](./integrations/01-github-integration.md) | GitHub in n8n — credentials, every field, 7 use cases incl. tracker deduplication, workflow backups |
| [`best-practices/`](./best-practices/) | [`01-best-practices.md`](./best-practices/01-best-practices.md) | Workflow design, webhook security, MCP, app mapping, credential management, performance |

---

## Quick Reference

### Hosting n8n — All Platforms
Every environment variable, every install method, Docker Compose annotated line-by-line, Cloud VPS with SSL.
→ [`hosting/complete-hosting-guide.md`](./hosting/complete-hosting-guide.md)

### Database Setup & Maintenance
- **SQLite vs PostgreSQL** — what each supports, when to switch
- **PostgreSQL** — Docker setup, manual Linux install, Supabase, all env variables, backup/restore, maintenance commands, migration from SQLite
- **MySQL/MariaDB** — deprecated for n8n backend (v1.0+), still usable in workflows
- **MongoDB, Redis, Supabase, NocoDB** — setup + n8n workflow examples
- **Decision guide** — which database to use for which situation
→ [`hosting/database-guide.md`](./hosting/database-guide.md)

### GitHub Integration
Tracker with deduplication, scheduled workflow backups, GitHub as config store, CI/CD triggers.
→ [`integrations/01-github-integration.md`](./integrations/01-github-integration.md)

### Node Field Guide
Every field in Telegram, Webhook, HTTP Request, Schedule, Code, If, Edit Fields, Google Sheets, Slack, Email, Wait, Merge, AI Agent, Loop — what it does, where to get the value, what breaks if empty.
→ [`nodes/05-node-fields-guide.md`](./nodes/05-node-fields-guide.md)

### AI Nodes & Local LLM Integration
Ollama step-by-step (4 scenarios), NVIDIA NIM, all open-source API providers, RAG pipelines.
→ [`nodes/03-ai-nodes-and-open-source-models.md`](./nodes/03-ai-nodes-and-open-source-models.md)

### Open-Source LLM Rate Limits
Groq, OpenRouter, NVIDIA, Hugging Face, Mistral, Cohere, DeepSeek, Grok, Gemini, Ollama hardware limits.
→ [`nodes/06-open-source-model-limits.md`](./nodes/06-open-source-model-limits.md)

---

*Maintained by [Shrinivas Ramaprasad](https://github.com/Shri-Phnx)*

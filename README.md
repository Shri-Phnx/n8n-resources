# n8n Resources

> **Author:** Shrinivas Ramaprasad
> A curated collection of n8n workflow automation references, cheat sheets, best practices, hosting guides, and integration patterns.
> **Node data source:** N8N_All_Nodes.xlsx — 819 nodes (67 core, 270 actions, 90 triggers, 21 root, 63 sub-nodes, 308 credentials)

---

## Folder Structure

```
n8n-resources/
├── keyboard-shortcuts/          Keyboard shortcuts — all 6 categories, Win + Mac
├── nodes/                       Node references, field guides, AI/LLM, limits
├── hosting/                     Complete hosting guide (all platforms, merged)
├── integrations/                Step-by-step integration patterns
└── best-practices/              Workflow design, security, performance
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
| [`hosting/`](./hosting/) | [`complete-hosting-guide.md`](./hosting/complete-hosting-guide.md) | **All-in-one hosting guide** — Windows/Mac/Linux npm, Docker, Cloud VPS, SSL, Postgres, Ollama stack, every env variable explained, backup, security |
| [`integrations/`](./integrations/) | [`01-github-integration.md`](./integrations/01-github-integration.md) | GitHub in n8n — credentials, every field, 7 use cases incl. tracker deduplication, scheduled backups, config store |
| [`best-practices/`](./best-practices/) | [`01-best-practices.md`](./best-practices/01-best-practices.md) | Workflow design, webhook security, MCP, app mapping, credential management, performance |

---

## Quick Reference

### Keyboard Shortcuts
→ [`keyboard-shortcuts/n8n-keyboard-shortcuts.md`](./keyboard-shortcuts/n8n-keyboard-shortcuts.md)

### Hosting n8n (All Methods — One File)
Every method, every field explained, every challenge solved:
- **Windows/Mac/Linux** — npm direct install, step-by-step
- **Docker** — single container, Docker Compose, with PostgreSQL, with Ollama
- **Cloud VPS** — Hetzner/DigitalOcean, nginx, SSL, domain setup, security hardening
- **All environment variables** — what each does, what breaks if you get it wrong
- **Backup & recovery**, update procedures, troubleshooting
→ [`hosting/complete-hosting-guide.md`](./hosting/complete-hosting-guide.md)

### GitHub Integration
Step-by-step GitHub integration with 7 use cases:
- Read/write files in GitHub from n8n workflows
- **Tracker with deduplication** — check GitHub before processing, write results back
- Scheduled workflow backup to private GitHub repo
- GitHub as configuration store
- Trigger n8n from GitHub events
- Limitations and better alternatives
→ [`integrations/01-github-integration.md`](./integrations/01-github-integration.md)

### Node Field Guide
Every field in Telegram, Webhook, HTTP Request, Schedule, Code, If, Edit Fields, Google Sheets, Slack, Email, Wait, Merge, AI Agent, Loop explained with what it does, where to get the value, and what breaks if you leave it empty.
→ [`nodes/05-node-fields-guide.md`](./nodes/05-node-fields-guide.md)

### Open-Source LLM Limits
Daily/hourly/per-minute limits for Groq, OpenRouter, NVIDIA NIM, Hugging Face, Mistral, Cohere, DeepSeek, Grok, Google Gemini, and Ollama hardware benchmarks.
→ [`nodes/06-open-source-model-limits.md`](./nodes/06-open-source-model-limits.md)

### AI Nodes & Local LLM Integration
All 21 AI root nodes, 28 LLM sub-nodes, Ollama step-by-step (4 scenarios: local, Docker, Linux Docker, n8n Cloud + ngrok), NVIDIA NIM, QNAP/NAS setup.
→ [`nodes/03-ai-nodes-and-open-source-models.md`](./nodes/03-ai-nodes-and-open-source-models.md)

---

*Maintained by [Shrinivas Ramaprasad](https://github.com/Shri-Phnx)*

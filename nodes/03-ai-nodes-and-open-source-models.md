# n8n AI Nodes, Local LLMs & Open-Source Model APIs — Complete Guide

> **Author:** Shrinivas Ramaprasad
> **Last updated:** April 2026
> **Node data source:** N8N_All_Nodes.xlsx (819 nodes — 67 core, 270 actions, 90 triggers, 21 root, 63 sub-nodes)

> ⚠️ = Known issue, common pitfall, or important warning. Always refer to official docs for latest API info.

---

## Overview

n8n has a native AI framework built on LangChain that enables building production-grade AI pipelines without writing LangChain code. Nodes are grouped into:

- **Root nodes** — orchestrate the AI pipeline (agent, chain, classifier, vector store)
- **Sub-nodes** — plug into root nodes (LLM models, memory, tools, vector stores, embeddings)

**Key principle:** Root nodes are the brain. Sub-nodes are the organs. You plug sub-nodes into the ports of the root node.

---

## Part 1 — Root Nodes (21 AI Orchestrators)

### AI Agent

The most powerful AI node — enables autonomous reasoning with access to tools.

| Parameter | Detail |
|-----------|--------|
| **Agent type** | Tools Agent (recommended), Conversational, ReAct, OpenAI Functions, Plan and Execute, SQL |
| **Prompt** | System message defining agent role and behaviour |
| **Require specific output format** | Force structured JSON output |
| **Max iterations** | Limit reasoning loops (default: 10 — increase for complex tasks) |
| **Return intermediate steps** | Debug: see tool calls and reasoning trace |

**How it works:**
1. Receives user input via Chat Trigger or Webhook
2. Reasons about which tool(s) to use
3. Calls tools (HTTP Request, Calculator, Workflow, etc.)
4. Incorporates results and reasons again
5. Repeats until a final answer is reached
6. Returns structured response

**Required sub-nodes:** At least one Chat Model. Tools and Memory are optional but strongly recommended.

**Example system prompt:**
```
You are an IT Asset Management assistant. You help users look up assets,
create incidents, and update records in ServiceNow.
Always confirm before making any changes.
Today's date is {{ $now.toFormat('dd MMMM yyyy') }}.
```

---

### Basic LLM Chain

Simplest AI pipeline — prompt in, LLM response out.

| Parameter | Detail |
|-----------|--------|
| **Prompt** | The message/prompt sent to the LLM |
| **Require specific output format** | JSON mode or schema enforcement |

**Required sub-nodes:** One Chat Model
**Use case:** Single-shot prompts, text classification, content generation, summarisation

---

### Question and Answer Chain (RAG)

| Parameter | Detail |
|-----------|--------|
| **Query** | User's question |
| **Include source documents** | Return which documents were used to answer |

**Required sub-nodes:** Chat Model + Vector Store Retriever
**Use case:** Chat with PDFs, internal knowledge bases, document Q&A

---

### Summarization Chain

| Mode | Detail |
|------|--------|
| **Map Reduce** | Summarise chunks, then summarise summaries — best for very long docs |
| **Stuff** | Pass all text at once (for shorter docs under the context limit) |
| **Refine** | Iteratively refine with each chunk — highest quality |

---

### Information Extractor

| Parameter | Detail |
|-----------|--------|
| **Schema** | Define fields to extract (name, type, description) |
| **Input text** | Text to extract from (email, document, message) |

**Use case:** Extract invoice fields from emails, parse CVs, structure customer feedback

---

### Text Classifier

| Parameter | Detail |
|-----------|--------|
| **Categories** | List of category names with optional descriptions |
| **Input text** | Text to classify |
| **Allow multiple classes** | Return multiple matching categories |
| **Fallback class** | Class returned if no match found |

**Use case:** Route support tickets, tag content, categorise emails by type

---

### Sentiment Analysis

| Output | Values |
|--------|--------|
| **Sentiment** | Positive, Negative, Neutral |
| **Confidence** | Score from 0 to 1 |

**Use case:** Analyse customer feedback, support tickets, reviews

---

### LangChain Code

| Attribute | Detail |
|-----------|--------|
| **Purpose** | Write custom LangChain pipelines directly in n8n |
| **Language** | JavaScript/TypeScript |
| **Use case** | Advanced AI flows not supported by built-in visual nodes |
| **⚠️ Note** | Requires LangChain knowledge; harder to maintain than visual nodes |

---

### Vector Store Root Nodes (12 options)

| Node | Backend | Hosted | Best For |
|------|---------|--------|----------|
| **Simple Vector Store** | In-memory | Local | Testing only — not persistent ⚠️ |
| **Chroma Vector Store** | Chroma | Self-hosted/cloud | Local AI apps |
| **Milvus Vector Store** | Milvus | Self-hosted/cloud | Large-scale systems |
| **MongoDB Atlas Vector Store** | MongoDB Atlas | Cloud | App backend AI |
| **PGVector Vector Store** | PostgreSQL | Self-hosted/cloud | SQL-based RAG |
| **Pinecone Vector Store** | Pinecone | Cloud | Production RAG |
| **Qdrant Vector Store** | Qdrant | Self-hosted/cloud | Search systems |
| **Redis Vector Store** | Redis | Self-hosted/cloud | Fast retrieval |
| **Supabase Vector Store** | Supabase | Cloud | Backend apps |
| **Weaviate Vector Store** | Weaviate | Self-hosted/cloud | Hybrid vector + keyword |
| **Zep Vector Store** | Zep | Self-hosted | Chat memory + vectors |
| **Azure AI Search** | Azure | Cloud | Enterprise Azure stack |

**Modes available on every vector store:**
- **Insert** — Add documents (build/index phase)
- **Retrieve** — Semantic search (query phase)
- **Retrieve as Tool** — Expose to AI Agent as a searchable tool

---

## Part 2 — Sub-nodes: LLM Chat Models (28 models)

> These plug into the **Chat Model** port of any root node.

### Paid / Proprietary Cloud Models

| Sub-node | Provider | Models | Free Tier |
|----------|---------|--------|-----------|
| **OpenAI Chat Model** | OpenAI | gpt-4o, gpt-4o-mini, o1, o3 | ❌ — ⚠️ Cost per token |
| **Anthropic Chat Model** | Anthropic | claude-opus-4-6, claude-sonnet-4-6, claude-haiku-4-5 | ❌ — ⚠️ Token limits |
| **Google Gemini Chat Model** | Google | gemini-2.0-flash, gemini-1.5-pro | ✅ Free tier |
| **Google Vertex Chat Model** | Google Cloud | gemini-pro | Limited |
| **AWS Bedrock Chat Model** | AWS | claude, llama, titan | Limited — ⚠️ Complex setup |
| **Azure OpenAI Chat Model** | Azure | GPT-4o (Azure-hosted) | ❌ — ⚠️ Setup required |
| **Vercel AI Gateway Chat Model** | Vercel | Multiple providers | ❌ |
| **Lemonade Chat Model** | Lemonade | Lemonade AI models | Paid |
| **Lemonade Model** | Lemonade | General LLM | Paid |

### Free / Open-Source / Low-Cost Models

| Sub-node | Provider | Models | Free Tier | Local? |
|----------|---------|--------|-----------|--------|
| **Ollama Chat Model** | Local (Ollama) | llama3, mistral, phi3, qwen, gemma, deepseek-r1 | ✅ 100% free | ✅ Yes |
| **Ollama Model** | Local (Ollama) | Any Ollama model (for non-chat chains) | ✅ 100% free | ✅ Yes |
| **Groq Chat Model** | Groq | llama-3.3-70b, mixtral, gemma2-9b | ✅ Free tier | ❌ Cloud |
| **OpenRouter Chat Model** | OpenRouter | 100+ models | ✅ Many free | ❌ Cloud |
| **DeepSeek Chat Model** | DeepSeek | deepseek-chat (V3), deepseek-reasoner (R1) | Very low cost | ❌ Cloud |
| **Mistral Cloud Chat Model** | Mistral AI | mistral-large, mistral-small, codestral | ✅ Experiment tier | ❌ Cloud |
| **Cohere Chat Model** | Cohere | command-r-plus, command-r | ✅ Free tier | ❌ Cloud |
| **xAI Grok Chat Model** | xAI | grok-3, grok-2 | Limited | ❌ Cloud |
| **Hugging Face Inference Model** | HuggingFace | 100k+ models | ✅ Rate-limited | Optional |

---

## Part 3 — Open-Source & Free API Providers

### Summary Comparison

| Provider | Free Tier | Local? | Best For | n8n Node |
|----------|-----------|--------|----------|----------|
| **Ollama** | ✅ Free forever | ✅ Yes | Privacy, offline, zero cost | Ollama Chat Model |
| **Groq** | ✅ Free tier | ❌ Cloud | Ultra-fast cloud inference | Groq Chat Model |
| **OpenRouter** | ✅ Many free models | ❌ Cloud | Access 100+ models from one API | OpenRouter Chat Model |
| **NVIDIA NIM** | ✅ Free credits | Optional | GPU-accelerated enterprise models | HTTP Request / community node |
| **Hugging Face** | ✅ Rate-limited | Optional | Research, niche models | HF Inference Model |
| **DeepSeek** | Very low cost | ❌ Cloud | Strong reasoning at low cost | DeepSeek Chat Model |
| **Mistral AI** | ✅ Experiment tier | ❌ Cloud | EU privacy-compliant option | Mistral Cloud Chat Model |
| **Cohere** | ✅ Free tier | ❌ Cloud | RAG + reranking + embeddings | Cohere Chat Model |
| **xAI Grok** | Limited | ❌ Cloud | Real-time knowledge access | xAI Grok Chat Model |
| **Google Gemini** | ✅ Free tier | ❌ Cloud | Multimodal, generous free quota | Google Gemini Chat Model |
| **ngrok** | ✅ Free tier | N/A | Expose local Ollama to n8n Cloud | Used alongside Ollama |

---

### API Key & URL Reference

| Provider | Sign Up | Base URL in n8n |
|----------|---------|----------------|
| **Groq** | [console.groq.com](https://console.groq.com) | Built into Groq node |
| **OpenRouter** | [openrouter.ai/keys](https://openrouter.ai/keys) | Built into OpenRouter node |
| **NVIDIA NIM** | [build.nvidia.com](https://build.nvidia.com) | `https://integrate.api.nvidia.com/v1` |
| **Hugging Face** | [huggingface.co/settings/tokens](https://huggingface.co/settings/tokens) | Built into HF node |
| **DeepSeek** | [platform.deepseek.com](https://platform.deepseek.com) | Built into DeepSeek node |
| **Mistral** | [console.mistral.ai](https://console.mistral.ai) | Built into Mistral node |
| **Cohere** | [dashboard.cohere.com](https://dashboard.cohere.com) | Built into Cohere node |
| **xAI (Grok)** | [console.x.ai](https://console.x.ai) | Built into xAI node |
| **Google Gemini** | [aistudio.google.com](https://aistudio.google.com) | Built into Gemini node |

---

### NVIDIA NIM — Detailed Integration

**What is NVIDIA NIM?**
NVIDIA NIM is a set of GPU-accelerated inference microservices that can run anywhere — cloud, data centre, workstation, or PC. It exposes an OpenAI-compatible API.

**NIM Tiers:**

| Tier | Description | Cost |
|------|-------------|------|
| **NIM Day 0** | Published within 72 hrs of model release | Free |
| **NIM Turbo** | Best-in-class performance on NVIDIA hardware | Free in production (early access) |
| **NIM Certified** | Enterprise: CVE SLAs, FIPS, long-term support | Requires NVIDIA AI Enterprise license |

**Popular models (as of April 2026):**
```
meta/llama-3.1-70b-instruct
meta/llama-3.1-405b-instruct
nvidia/llama-3.1-nemotron-70b-instruct
nvidia/llama-3.3-nemotron-super-49b-v1
meta/llama-3.2-11b-vision-instruct       ← vision/multimodal
mistralai/mistral-large-2-instruct
google/gemma-2-27b-it
microsoft/phi-3.5-mini-instruct
```

**Method A — Community node (recommended for quick start):**

Package: `n8n-nodes-nvidia-nim` (v2.3.4, Feb 2026)

1. **Settings → Community Nodes → Install**
2. Enter `n8n-nodes-nvidia-nim` → accept risk → **Install**
3. Get API key from [build.nvidia.com](https://build.nvidia.com)
4. Create **NVIDIA NIM credential** → paste key → Save
5. Add **NVIDIA NIM Chat** node → select model → configure prompt

Nodes added: `NVIDIA NIM Chat` and `NVIDIA NIM Image Analysis`

**Method B — HTTP Request node:**
```
Method: POST
URL: https://integrate.api.nvidia.com/v1/chat/completions
Headers: Authorization: Bearer <YOUR_NVIDIA_API_KEY>
Body:
{
  "model": "meta/llama-3.1-70b-instruct",
  "messages": [{"role": "user", "content": "{{ $json.prompt }}"}],
  "max_tokens": 1024,
  "temperature": 0.7
}
Extract response: {{ $json.choices[0].message.content }}
```

**Method C — Via OpenRouter (no separate NVIDIA account needed):**
1. Sign up at [openrouter.ai](https://openrouter.ai)
2. Add **OpenRouter Chat Model** sub-node
3. Model string: `nvidia/llama-3.1-nemotron-70b-instruct`

**Method D — Self-hosted NIM container (for enterprise):**
```bash
docker run -it --rm --gpus all --shm-size=16GB \
  -e NGC_API_KEY=$NGC_API_KEY \
  -v "$LOCAL_NIM_CACHE:/opt/nim/.cache" \
  -p 8000:8000 \
  nvcr.io/nim/meta/llama-3.1-8b-instruct:latest

# In n8n HTTP Request: URL: http://your-gpu-server:8000/v1/chat/completions
```

---

### Groq — Free Ultra-Fast Inference

Free tier includes 30 req/min, 6,000 tokens/min, 14,400 req/day.

**How to connect:**
1. Sign up at [console.groq.com](https://console.groq.com)
2. **API Keys → Create API Key** → copy
3. In n8n: Add **Groq Chat Model** sub-node
4. **Credential → Create new** → paste key → **Save**
5. Select model: `llama-3.3-70b-versatile`, `mixtral-8x7b-32768`, etc.

**⚠️ Rate limit hit:** Add a **Wait** node (5 seconds) before the Groq node to throttle requests.

---

### Hugging Face Inference API

**How to connect:**
1. Sign up at [huggingface.co](https://huggingface.co)
2. **Settings → Access Tokens → New Token** → Read role → copy
3. In n8n: Add **Hugging Face Inference Model** sub-node
4. **Credential → Create new** → paste token → **Save**
5. Enter model ID (e.g., `meta-llama/Meta-Llama-3-8B-Instruct`)

**⚠️ "Model is currently loading"** → Model is cold-starting. Wait 30–60 seconds and retry. Add a **Wait** node (30s) before the HF node for automation.

---

## Part 4 — Local LLM Integration (Step-by-Step)

### Why Run a Local LLM?

| Reason | Benefit |
|--------|---------|
| **Privacy** | Data never leaves your machine or server |
| **Cost** | Zero API costs after initial hardware setup |
| **Offline** | Works without internet connection |
| **No rate limits** | Process as many tokens as hardware allows |
| **Compliance** | HIPAA, GDPR, SOC 2 — no third-party data exposure |

---

## 🥇 Method 1 — Ollama (RECOMMENDED — Start Here)

> Ollama is the recommended first choice. Simplest setup, best n8n integration, most widely documented.

### What is Ollama?

An open-source tool (think "Docker for AI models") that:
- Downloads and runs LLMs locally with one command
- Exposes an OpenAI-compatible REST API at `http://localhost:11434`
- Automatically handles GPU/CPU, quantization, and memory management
- Supports 100+ models (Llama, Mistral, Phi, Gemma, DeepSeek, Qwen, etc.)

### Hardware Requirements

| Model Size | Minimum RAM | Recommended |
|------------|-------------|-------------|
| 1–4B parameters | 4 GB RAM | No GPU needed |
| 7–8B parameters | 8 GB RAM | 6–8 GB VRAM |
| 13B parameters | 16 GB RAM | 12 GB VRAM |
| 30–34B parameters | 32 GB RAM | 24 GB VRAM |
| 70B parameters | 64 GB RAM | 48 GB VRAM |

> ℹ️ Quantized models (Q4, Q5, Q8) use significantly less RAM. Llama 3.1 8B Q4 runs on 8 GB RAM with no GPU.

---

### Step 1 — Install Ollama

**Windows:**
1. Go to [https://ollama.com/download](https://ollama.com/download)
2. Click **Download for Windows** → run the `.exe` installer
3. Ollama installs and starts automatically as a background service
4. Verify: Open Command Prompt → type `ollama --version`

**macOS:**
```bash
# Option A — Download from website
# ollama.com → Download for macOS → open .dmg → drag to Applications

# Option B — Homebrew
brew install ollama
```

**Linux (Ubuntu/Debian/RHEL):**
```bash
curl -fsSL https://ollama.com/install.sh | sh
# Installs Ollama as a systemd service that starts automatically on boot
```

**Verify installation:**
```bash
ollama --version
# Expected output: ollama version 0.x.x
```

**⚠️ Common install issues:**
- **Windows: "ollama not found"** → Restart Command Prompt after install; or add `C:\Users\<you>\AppData\Local\Programs\Ollama` to your system PATH
- **macOS: "app can't be opened" security warning** → System Settings → Privacy & Security → "Open Anyway"
- **Linux: `curl` not found** → `sudo apt install curl` first

---

### Step 2 — Download a Model

```bash
# General purpose (3B, great for low-spec machines — needs 4 GB RAM)
ollama pull llama3.2:3b

# Balanced quality (8B — needs 8 GB RAM)
ollama pull llama3.1:8b

# Fast Microsoft model (14B, strong reasoning)
ollama pull phi4:14b

# Reasoning-focused (7B)
ollama pull deepseek-r1:7b

# Long context support (128K tokens)
ollama pull qwen2.5:7b

# For embeddings (RAG pipelines)
ollama pull nomic-embed-text

# Check what you have installed
ollama list

# Remove a model to free disk space
ollama rm llama3.2:3b
```

**⚠️ Common download issues:**
- **Download stalls or stops** → Normal — models are 2–40 GB. Press Ctrl+C and re-run `ollama pull` — it resumes from where it stopped
- **"Error: model not found"** → Check exact model name at [ollama.com/library](https://ollama.com/library) — names are case-sensitive
- **"Insufficient disk space"** → Models are stored in `~/.ollama/models` (Linux/Mac) or `C:\Users\<you>\.ollama\models` (Windows). Free disk space or point to another drive:
  ```bash
  # Linux/Mac: set custom model path
  export OLLAMA_MODELS=/path/to/large/drive/ollama-models
  ```

---

### Step 3 — Verify Ollama is Running

```bash
# Test the Ollama REST API
curl http://localhost:11434
# Expected: "Ollama is running"

# Test a model via API
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2:3b",
  "prompt": "Hello! Are you running?",
  "stream": false
}'
# Expected: {"model":"llama3.2:3b", "response":"Hello! Yes, I am running...", ...}
```

**If Ollama is not running:**

*Windows:* Search "Ollama" in Start Menu → open it. Alternatively, look for the llama icon in the system tray (bottom-right corner).

*macOS:* Open Ollama from Applications folder. Look for the llama icon in the menu bar.

*Linux:*
```bash
sudo systemctl start ollama
sudo systemctl status ollama    # Check if active (running)
sudo systemctl enable ollama    # Auto-start on boot
```

---

### Step 4 — Connect Ollama to n8n

**There are 4 scenarios — choose the one that matches your setup:**

---

#### Scenario A — n8n and Ollama both on the same machine (not Docker)

This is the simplest case. Both apps share the same `localhost`.

1. Open n8n in your browser (`http://localhost:5678`)
2. Create a new workflow
3. Add a **Basic LLM Chain** root node (or AI Agent, Q&A Chain, etc.)
4. Click the **Chat Model** sub-node port → search **Ollama Chat Model** → click it
5. Click **Select Credential → Create new credential**
6. **Base URL:** `http://localhost:11434` (try `http://127.0.0.1:11434` if this fails)
7. Click **Test** → wait 5 seconds → should show **"Connection tested successfully"** ✅
8. Close credential → in the node, click **Model** dropdown → select your model (e.g., `llama3.2:3b`)
9. If Model dropdown is empty: refresh browser tab → reopen the node → it should now populate

**⚠️ Connection fails on localhost:**
- Try `http://127.0.0.1:11434` instead — resolves DNS differently
- Run `curl http://localhost:11434` in terminal — if this also fails, Ollama is not running; start it
- Windows Defender/Firewall may block port 11434 → Windows Defender Firewall → Allow an app → add Ollama

---

#### Scenario B — n8n in Docker, Ollama on the host machine

Docker containers cannot use `localhost` to reach the host machine — it points to the container itself. You must use a special hostname.

**Sub-scenario B1 — Docker Desktop (Windows/macOS):**

Docker Desktop automatically provides `host.docker.internal` to reach the host.

In the Ollama Chat Model credential:
```
Base URL: http://host.docker.internal:11434
```
Click Test → should show ✅

**Sub-scenario B2 — Docker Engine on Linux (without Docker Desktop):**

`host.docker.internal` is NOT available by default on Linux Docker Engine.

**Fix — add `extra_hosts` in docker-compose.yml:**
```yaml
services:
  n8n:
    image: n8nio/n8n
    extra_hosts:
      - "host.docker.internal:host-gateway"
    ports:
      - "5678:5678"
    environment:
      - N8N_SECURE_COOKIE=false
```

Restart n8n:
```bash
docker compose down && docker compose up -d
```

Now use `http://host.docker.internal:11434` as Base URL.

**Alternative — use your host machine's LAN IP:**
```bash
# Find your host LAN IP
ip route | grep default | awk '{print $3}'
# Or: hostname -I | awk '{print $1}'
# Example output: 192.168.1.105
```
Base URL: `http://192.168.1.105:11434`

> ⚠️ Also ensure Ollama listens on all interfaces (default is localhost only):
```bash
# Set before starting Ollama
export OLLAMA_HOST=0.0.0.0
ollama serve

# Or permanently in systemd (Linux):
sudo systemctl edit ollama
# Add under [Service]:
# Environment="OLLAMA_HOST=0.0.0.0"
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

---

**Sub-scenario B3 — n8n and Ollama both in Docker (docker-compose):**
```yaml
services:
  n8n:
    image: n8nio/n8n
    ports:
      - "5678:5678"
    depends_on:
      - ollama

  ollama:
    image: ollama/ollama
    ports:
      - "11434:11434"
    volumes:
      - ollama_data:/root/.ollama

volumes:
  ollama_data:
```

Pull a model after starting the containers:
```bash
docker exec -it ollama ollama pull llama3.1:8b
```

In n8n Ollama credential:
```
Base URL: http://ollama:11434
```
(Uses the Docker service name as the hostname — Docker's internal DNS resolves it automatically)

---

#### Scenario C — n8n Cloud, Ollama on your local machine (ngrok tunnel)

n8n Cloud is hosted on the internet. Your local Ollama is behind your home/office router. n8n Cloud cannot reach `localhost` — you need a secure public tunnel.

**Step 1 — Install ngrok:**
- Go to [ngrok.com](https://ngrok.com) → sign up (free)
- Download for your OS
- *Windows:* run the `.exe`; *Mac:* `brew install ngrok`; *Linux:* `snap install ngrok`

**Step 2 — Authenticate:**
```bash
ngrok config add-authtoken <YOUR_NGROK_TOKEN>
# Token found at: dashboard.ngrok.com/get-started/your-authtoken
```

**Step 3 — Make sure Ollama listens on all interfaces:**
```bash
# Mac/Linux — set before starting Ollama
OLLAMA_HOST=0.0.0.0 ollama serve

# Windows — in Command Prompt before opening the Ollama app
set OLLAMA_HOST=0.0.0.0
```

**Step 4 — Start the ngrok tunnel:**
```bash
ngrok http 11434
# Output shows something like:
# Forwarding   https://abc123.ngrok-free.app -> http://localhost:11434
```

**Step 5 — In n8n Cloud Ollama credential:**
```
Base URL: https://abc123.ngrok-free.app
```
Click Test → should connect ✅

**Step 6 — Select your model** in the Ollama Chat Model node.

**⚠️ ngrok issues:**
- **Free tier: URL changes every restart** → Re-paste the new URL into n8n credential each time. For a permanent URL, use ngrok paid plan or Cloudflare Tunnel (free)
- **"ERR_NGROK_8012: Browser warning page"** → ngrok adds a browser warning for free tunnels. Fix: add header `ngrok-skip-browser-warning: true` in the Ollama credential (under Custom Headers)
- **Ollama still refuses connections** → Ensure `OLLAMA_HOST=0.0.0.0` is set BEFORE Ollama starts

---

#### Scenario D — Ollama on a separate server or NAS (e.g., QNAP TS-x73A)

```bash
# Install Ollama on QNAP via Container Station
docker run -d --name ollama \
  -p 11434:11434 \
  -v /share/ollama:/root/.ollama \
  -e OLLAMA_HOST=0.0.0.0 \
  ollama/ollama

# Pull a model on the NAS
docker exec -it ollama ollama pull llama3.1:8b

# In n8n Ollama credential Base URL:
http://<NAS_IP_ADDRESS>:11434
# Example: http://192.168.1.100:11434
```

**⚠️ NAS/server issues:**
- **Timeout connecting** → Verify port 11434 is open in your NAS firewall settings and your LAN router allows traffic between devices
- **Slow responses** → CPU-only inference on NAS is slow. Use a small quantized model: `llama3.2:3b` or `phi3:mini`
- **QNAP VLAN isolation** → Check that n8n and NAS are on the same VLAN or that cross-VLAN routing is enabled

---

### Step 5 — Troubleshooting Ollama in n8n (Complete Reference)

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Model dropdown is empty | No models installed | Run `ollama pull llama3.1:8b` in terminal |
| "Connection tested successfully" but model list empty | Browser/n8n cache | Refresh browser tab → reopen node → should populate |
| Very slow responses | CPU-only, model too large | Switch to smaller quantized model (3B or 7B Q4) |
| Workflow times out | Model cold start delay | Pre-load: `ollama run llama3.1:8b` before running workflow; increase n8n HTTP timeout in Settings |
| "Context length exceeded" error | Input too long for model | Use Recursive Character Text Splitter + Summarization Chain first |
| Docker: "connection refused" | Using localhost in Docker | Use `host.docker.internal:11434`; add `extra_hosts` for Linux |
| Docker on Linux: host.docker.internal unknown | Not supported on Linux by default | Add `extra_hosts: ["host.docker.internal:host-gateway"]` to docker-compose.yml |
| Model responses garbled or wrong format | Model not optimised for chat format | Try a different model; some models need specific prompt templates |
| Ollama crashes under load | RAM exhausted | Use smaller model; add swap: `sudo fallocate -l 8G /swapfile` |
| Cannot reach Ollama from another machine | Ollama only listening on localhost | Set `OLLAMA_HOST=0.0.0.0` before starting |
| ngrok warning page blocks API calls | Free ngrok browser warning | Add header `ngrok-skip-browser-warning: true` in credential |
| "OLLAMA_HOST not recognized" (Windows) | Environment variable not set | Set in System Properties → Environment Variables, then restart |

---

### Step 6 — Build Your First Local AI Workflow

**Minimal working workflow — local AI chatbot:**
```
Chat Trigger
    └── Basic LLM Chain
          └── [Chat Model] Ollama Chat Model
                          Credential: Ollama (http://localhost:11434)
                          Model: llama3.2:3b
```

**Full agent with memory and tools:**
```
Chat Trigger
    └── AI Agent
          ├── [Chat Model] Ollama Chat Model (llama3.1:8b)
          ├── [Memory]     Simple Memory (Session ID: {{ $sessionId }})
          └── [Tools]      Calculator + Wikipedia
```

**System prompt:**
```
You are a helpful IT assistant. You help users troubleshoot IT issues.
Always ask a clarifying question before suggesting solutions.
Today: {{ $now.toFormat('dd MMMM yyyy') }}.
```

---

### Step 7 — Recommended Models by Use Case

| Use Case | Model | Command | RAM Required |
|----------|-------|---------|-------------|
| Low-spec machine | phi3:mini | `ollama pull phi3:mini` | 4 GB |
| General chat | llama3.2:3b | `ollama pull llama3.2:3b` | 4 GB |
| Better quality | llama3.1:8b | `ollama pull llama3.1:8b` | 8 GB |
| Long context (128K) | qwen2.5:7b | `ollama pull qwen2.5:7b` | 8 GB |
| Coding + reasoning | deepseek-r1:7b | `ollama pull deepseek-r1:7b` | 8 GB |
| High quality | phi4:14b | `ollama pull phi4:14b` | 16 GB |
| Vision/multimodal | llava:7b | `ollama pull llava:7b` | 8 GB |
| Embeddings (RAG) | nomic-embed-text | `ollama pull nomic-embed-text` | 1 GB |

---

## 🥈 Method 2 — LM Studio (Best GUI Experience)

**Best for:** Non-developers, researchers, users who prefer a visual model browser.

| Attribute | Detail |
|-----------|--------|
| **What it is** | Desktop app for running LLMs with built-in chat UI |
| **Platforms** | Windows, macOS, Linux |
| **API** | OpenAI-compatible at `http://localhost:1234/v1` |
| **Download** | [lmstudio.ai](https://lmstudio.ai) |

**Steps:**
1. Download and install from [lmstudio.ai](https://lmstudio.ai)
2. Open LM Studio → go to **Discover** tab → search for model → **Download**
3. Go to **Developer** tab (or **Local Server**) → select downloaded model → click **Start Server**
4. Note the URL: `http://localhost:1234/v1`

**Connect to n8n — use OpenAI Chat Model sub-node with custom base URL:**

Create a new OpenAI credential with:
```
API Key: lm-studio    (any string — LM Studio doesn't validate it)
Base URL: http://localhost:1234/v1
```
Select model: type the model name shown in LM Studio (e.g., `lmstudio-community/Meta-Llama-3-8B-Instruct-GGUF`)

**⚠️ LM Studio issues:**
- **n8n can't find model** → LM Studio must have a model LOADED in the Chat tab AND the server started in the Developer tab
- **Docker n8n → LM Studio** → Use `host.docker.internal:1234` (same logic as Ollama)
- **Slow first response** → LM Studio loads model on first request — wait 30-60 seconds

---

## 🥉 Method 3 — GPT4All (Simplest All-in-One)

**Best for:** Complete beginners — no command line required.

| Attribute | Detail |
|-----------|--------|
| **What it is** | Desktop app with integrated model browser and local server |
| **Platforms** | Windows, macOS, Linux |
| **API** | OpenAI-compatible at `http://localhost:4891/v1` |
| **Download** | [gpt4all.io](https://gpt4all.io) |

**Steps:**
1. Download and install from [gpt4all.io](https://gpt4all.io)
2. Open GPT4All → **Models** tab → download a model
3. **Settings → API Server → Enable**
4. In n8n: use OpenAI Chat Model credential:
   ```
   API Key: not-needed
   Base URL: http://localhost:4891/v1
   ```

---

## Part 5 — Sub-nodes: Memory, Embeddings, Output Parsers & Tools

### Memory Sub-nodes

| Node | Persistent | Use Case | ⚠️ Warning |
|------|-----------|----------|------------|
| **Simple Memory** | ❌ Session only | Testing, demos | Lost on n8n restart ⚠️ |
| **Redis Chat Memory** | ✅ Yes | Scalable production | Redis volatile unless persistence configured ⚠️ |
| **Postgres Chat Memory** | ✅ Yes | SQL-based stacks | Needs DB setup |
| **MongoDB Chat Memory** | ✅ Yes | Document stacks | Needs DB setup |
| **Motorhead** | ✅ Yes | Persistent memory backend | Requires external Motorhead server |
| **Xata** | ✅ Yes | Cloud DB memory | API limits |
| **Zep** | ✅ Yes | Long-term memory + vectors | Requires self-hosted Zep |

**Session ID tip:** Use `{{ $sessionId }}` from Chat Trigger to group messages by conversation automatically.

---

### Embeddings Sub-nodes

| Node | Provider | Free? | Best For |
|------|---------|-------|---------|
| **Embeddings Ollama** | Local | ✅ Free | Local RAG (use with nomic-embed-text) |
| **Embeddings HuggingFace Inference** | HuggingFace | ✅ Free tier | Research |
| **Embeddings OpenAI** | OpenAI | ❌ Paid | Production accuracy — ⚠️ cost |
| **Embeddings Google Gemini** | Google | ✅ Free tier | Google stack |
| **Embeddings Mistral Cloud** | Mistral | ✅ Free tier | EU-compliant |
| **Embeddings Cohere** | Cohere | ✅ Free tier | RAG + reranking |
| **Embeddings Azure OpenAI** | Azure | ❌ Paid | Azure stack — ⚠️ config required |
| **Embeddings AWS Bedrock** | AWS | Limited | AWS stack — ⚠️ complex setup |
| **Embeddings Google PaLM** | Google | ⚠️ Deprecated | Migrate to Gemini |

---

### Output Parser Sub-nodes

| Node | Purpose | ⚠️ Notes |
|------|---------|---------|
| **Structured Output Parser** | Enforce JSON schema on LLM output | Fails if LLM doesn't comply — use with reliable models ⚠️ |
| **Auto-fixing Output Parser** | Retry and fix malformed LLM output | Not 100% reliable — validate downstream ⚠️ |
| **Item List Output Parser** | Parse comma-separated lists from output | Needs consistent LLM output format |

---

### Tool Sub-nodes (for AI Agent)

| Tool | What the Agent Can Do | Notes |
|------|-----------------------|-------|
| **HTTP Request Tool** | Call any external REST/GraphQL API | Validate URL and auth |
| **Calculator** | Mathematical operations | Numeric only |
| **Wikipedia** | Search Wikipedia articles | Public data |
| **SerpApi (Google Search)** | Live Google searches | ⚠️ Paid |
| **SearXNG Tool** | Privacy search engine | ⚠️ Self-hosted required |
| **Wolfram Alpha** | Advanced computation | ⚠️ Paid |
| **Vector Store Q&A Tool** | Semantic search in vector DB | Needs vector store configured |
| **Custom Code Tool** | Custom JS/Python logic | ⚠️ Security risk — validate input |
| **Call n8n Workflow Tool** | Invoke another n8n workflow | Requires target workflow |
| **AI Agent Tool** | Nest another agent as sub-agent | Needs configured agent |
| **MCP Client Tool** | Connect to MCP services | ⚠️ Setup required |
| **Think Tool** | Chain-of-thought reasoning | ⚠️ Experimental |

---

### Retriever Sub-nodes

| Node | Purpose | ⚠️ Notes |
|------|---------|---------|
| **Vector Store Retriever** | Retrieve semantically similar docs | Needs embeddings configured |
| **Contextual Compression Retriever** | Compress and filter retrieved docs | Adds complexity |
| **MultiQuery Retriever** | Run multiple query variants for better recall | ⚠️ Higher cost — multiple LLM calls |
| **Workflow Retriever** | Retrieve data via another workflow | Limited use cases |

---

## Part 6 — Complete Workflow Examples

### Example 1 — Fully Local IT Helpdesk Agent

```
Chat Trigger
    └── AI Agent
          ├── [Model]   Ollama Chat Model
          │             Model: llama3.1:8b
          │             Credential: http://localhost:11434
          ├── [Memory]  Postgres Chat Memory
          │             Session: {{ $sessionId }}
          └── [Tools]
                ├── HTTP Request Tool → ServiceNow REST API
                └── Call Workflow Tool → "Send Email" workflow
```

System prompt:
```
You are an IT helpdesk assistant. You can look up assets in ServiceNow.
Always confirm before creating or modifying any record.
Date: {{ $now.toFormat('dd MMM yyyy') }}.
```

---

### Example 2 — 100% Local RAG Pipeline

**Index phase:**
```
Manual Trigger → Read Files → Default Data Loader
    → Recursive Character Text Splitter
    → PGVector Vector Store [Insert]
          └── Embeddings Ollama (nomic-embed-text)
```

**Query phase:**
```
Webhook → Q&A Chain
    ├── [Model]     Ollama Chat Model (llama3.1:8b)
    └── [Retriever] Vector Store Retriever
          └── PGVector [Retrieve] + Embeddings Ollama
→ Respond to Webhook
```

---

### Example 3 — Multi-Model Routing

```
Chat Trigger
    → If (complex task?)
          [True]  → Basic LLM Chain → Ollama (local, no cost)
          [False] → Basic LLM Chain → Groq (fast, free tier)
    → Merge → Respond to Webhook
```

---

## Part 7 — Decision Guide: Which LLM Provider to Choose?

```
Do you have privacy/compliance requirements?
│
├── YES → Local LLM
│          Ollama (recommended — see Part 4)
│          LM Studio (GUI preference)
│          Choose model by RAM:
│              < 8 GB  → llama3.2:3b or phi3:mini
│              8 GB    → llama3.1:8b or mistral:7b
│              16 GB+  → phi4:14b or qwen2.5:14b
│
└── NO → Cloud API
         ├── Need speed?          → Groq (fastest free tier)
         ├── Need free?           → Groq or Google Gemini free tier
         ├── Need most capable?   → OpenAI GPT-4o or Anthropic Claude
         ├── Need cheapest?       → DeepSeek (lowest cost/token)
         ├── Need 100+ models?    → OpenRouter (one API key)
         ├── Need EU data residency? → Mistral AI
         └── Need NVIDIA GPU models? → NVIDIA NIM (build.nvidia.com)
```

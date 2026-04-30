# n8n AI Nodes, Local LLMs & Open-Source Model APIs — Complete Guide

> **Author:** Shrinivas Ramaprasad  
> **Last updated:** April 2026  
> **Source:** n8n Docs, N8N_All_Nodes.xlsx, Ollama Docs, provider official docs

> ⚠️ = Known issue, common pitfall, or important warning.

---

## Overview

n8n has a native AI framework built on LangChain that enables building production-grade AI pipelines without writing LangChain code. Nodes are grouped into:

- **Root nodes** — orchestrate the AI pipeline (agent, chain, classifier, vector store)
- **Sub-nodes** — plug into root nodes (LLM models, memory, tools, vector stores, embeddings)

> **Key principle:** Root nodes are the brain. Sub-nodes are the organs. You plug sub-nodes into the ports on the root node.

---

## Part 1 — Root Nodes (21 AI Orchestrators)

### AI Agent

The most powerful AI node — enables autonomous reasoning with access to tools.

| Parameter | Detail |
|-----------|--------|
| **Agent type** | Tools Agent (recommended), Conversational, ReAct, OpenAI Functions, Plan and Execute, SQL |
| **Prompt** | System message defining agent role and behaviour |
| **Require specific output format** | Force structured JSON output |
| **Max iterations** | Limit reasoning loops (default: 10) |
| **Return intermediate steps** | Debug — see tool calls and reasoning |

**How it works:**
1. Receives user input via Chat Trigger or Webhook
2. Reasons about which tool(s) to use
3. Calls tools (HTTP Request, Calculator, Workflow, etc.)
4. Incorporates tool results into reasoning
5. Repeats until final answer is reached
6. Returns final response

**Required sub-nodes:** At least one Chat Model. Tools and Memory are optional but strongly recommended.

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

### Question and Answer Chain

RAG pipeline — answers questions from a document store.

| Parameter | Detail |
|-----------|--------|
| **Query** | User's question |
| **Include source documents** | Return which documents were used |

**Required sub-nodes:** Chat Model + Vector Store Retriever  
**Use case:** Chat with PDFs, internal knowledge bases, document Q&A

---

### Summarization Chain

| Mode | Detail |
|------|--------|
| **Map Reduce** | Summarise chunks, then summarise summaries (best for long docs) |
| **Stuff** | Pass all text at once (for shorter docs) |
| **Refine** | Iteratively refine summary with each chunk |

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
| **Fallback class** | Class if no match found |

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
| **Use case** | Advanced AI flows not supported by visual nodes |
| **⚠️ Note** | Requires LangChain knowledge; less maintainable than visual nodes |

---

### Vector Store Nodes (9 options)

| Node | Backend | Hosted | Best For |
|------|---------|--------|----------|
| **Simple Vector Store** | In-memory | ✅ | Testing only — not persistent |
| **Chroma Vector Store** | Chroma | Self-hosted or cloud | Local AI apps |
| **Milvus Vector Store** | Milvus | Self-hosted or cloud | Large-scale AI systems |
| **MongoDB Atlas Vector Store** | MongoDB Atlas | Cloud | App backend AI |
| **PGVector Vector Store** | PostgreSQL | Self-hosted or cloud | SQL-based RAG |
| **Pinecone Vector Store** | Pinecone | Cloud | Production RAG |
| **Qdrant Vector Store** | Qdrant | Self-hosted or cloud | Search systems |
| **Redis Vector Store** | Redis | Self-hosted or cloud | Fast retrieval |
| **Supabase Vector Store** | Supabase | Cloud | Backend apps |
| **Weaviate Vector Store** | Weaviate | Self-hosted or cloud | Hybrid search |
| **Zep Vector Store** | Zep | Self-hosted | Chat memory + vectors |
| **Azure AI Search** | Azure | Cloud | Enterprise Azure stack |

**Modes:**
- **Insert** — Add documents (build/index phase)
- **Retrieve** — Search similar documents (query phase)
- **Retrieve as Tool** — Expose to AI Agent

---

## Part 2 — Sub-nodes: LLM Chat Models

> These plug into the **Chat Model** port of any Root node.

### Paid / Cloud API Models

| Sub-node | Provider | Models | Free Tier? |
|----------|---------|--------|------------|
| **Anthropic Chat Model** | Anthropic | claude-opus-4-6, claude-sonnet-4-6, claude-haiku-4-5 | ❌ |
| **AWS Bedrock Chat Model** | AWS | claude, llama, titan, mistral | Limited |
| **Azure OpenAI Chat Model** | Azure | GPT-4o, GPT-4, etc. | ❌ |
| **Google Gemini Chat Model** | Google | gemini-2.0-flash, gemini-1.5-pro | ✅ Free tier |
| **Google Vertex Chat Model** | Google Cloud | gemini-pro, palm | Limited |
| **OpenAI Chat Model** | OpenAI | gpt-4o, gpt-4o-mini, o1, o3 | ❌ |
| **Vercel AI Gateway Chat Model** | Vercel | Multiple providers | ❌ |

### Free / Open-Source API Models

| Sub-node | Provider | Models | Free Tier? |
|----------|---------|--------|------------|
| **Cohere Chat Model** | Cohere | command-r, command-r-plus | ✅ Free tier |
| **DeepSeek Chat Model** | DeepSeek | deepseek-chat (V3), deepseek-reasoner (R1) | ✅ Very low cost |
| **Groq Chat Model** | Groq | llama-3.3-70b, mixtral, gemma2-9b | ✅ Free tier |
| **Mistral Cloud Chat Model** | Mistral AI | mistral-large, mistral-small | ✅ Free tier |
| **Ollama Chat Model** | Local (Ollama) | llama3, mistral, phi3, qwen2.5, gemma3 | ✅ Free (local) |
| **OpenRouter Chat Model** | OpenRouter | 100+ models incl. NVIDIA, Llama, Mistral | ✅ Free models available |
| **xAI Grok Chat Model** | xAI | grok-3, grok-2 | Limited |

---

## Part 3 — Open-Source & Free API Providers

### Provider Comparison

| Provider | Free Tier | Local? | API Standard | Best For | n8n Node |
|----------|-----------|--------|-------------|----------|----------|
| **Ollama** | ✅ Free forever | ✅ Yes | OpenAI-compatible | Privacy, offline, no cost | Ollama Chat Model |
| **Groq** | ✅ Free tier | ❌ Cloud | OpenAI-compatible | Ultra-fast cloud inference | Groq Chat Model |
| **OpenRouter** | ✅ Free models | ❌ Cloud | OpenAI-compatible | Access 100+ models from 1 API | OpenRouter Chat Model |
| **NVIDIA NIM** | ✅ Free (build.nvidia.com) | Optional | OpenAI-compatible | GPU-accelerated enterprise models | HTTP Request / community node |
| **Hugging Face** | ✅ Free tier | Optional | HF API | Research, niche models | HF Inference Model sub-node |
| **DeepSeek** | ✅ Very cheap | ❌ Cloud | OpenAI-compatible | Strong reasoning at low cost | DeepSeek Chat Model |
| **Mistral AI** | ✅ Free tier | ❌ Cloud | OpenAI-compatible | European privacy-compliant option | Mistral Cloud Chat Model |
| **Cohere** | ✅ Free tier | ❌ Cloud | Cohere API | RAG and embeddings | Cohere Chat Model |
| **xAI (Grok)** | Limited | ❌ Cloud | OpenAI-compatible | Latest Grok models | xAI Grok Chat Model |
| **Google Gemini** | ✅ Free tier | ❌ Cloud | Google API | Multimodal, free quota | Google Gemini Chat Model |
| **ngrok** | ✅ Free tier | N/A | Tunnel | Expose local Ollama to n8n Cloud | Used with Ollama |

---

### Provider Setup — API Keys & URLs

| Provider | Get API Key | Base URL |
|----------|------------|----------|
| **Groq** | [console.groq.com](https://console.groq.com) | Built into n8n node |
| **OpenRouter** | [openrouter.ai/keys](https://openrouter.ai/keys) | Built into n8n node |
| **NVIDIA NIM** | [build.nvidia.com](https://build.nvidia.com) | `https://integrate.api.nvidia.com/v1` |
| **Hugging Face** | [huggingface.co/settings/tokens](https://huggingface.co/settings/tokens) | Built into n8n node |
| **DeepSeek** | [platform.deepseek.com](https://platform.deepseek.com) | Built into n8n node |
| **Mistral** | [console.mistral.ai](https://console.mistral.ai) | Built into n8n node |
| **Cohere** | [dashboard.cohere.com](https://dashboard.cohere.com) | Built into n8n node |
| **xAI (Grok)** | [console.x.ai](https://console.x.ai) | Built into n8n node |
| **Google Gemini** | [aistudio.google.com](https://aistudio.google.com) | Built into n8n node |

---

### NVIDIA NIM — Detailed Integration

#### What is NVIDIA NIM?

NVIDIA NIM is a set of accelerated inference microservices that allow organisations to run AI models on NVIDIA GPUs anywhere — cloud, data centre, workstations, and PCs. It exposes an **OpenAI-compatible API**.

#### NIM Tiers

| Tier | Description | Cost |
|------|-------------|------|
| **NIM Day 0** | Published within 72 hrs of model release | Free |
| **NIM Turbo** | Best-in-class performance (early access) | Free in production |
| **NIM Certified** | Enterprise: CVE SLAs, FIPS, long-term support | Requires NVIDIA AI Enterprise license |

#### Method 1 — Community Node (Recommended)

**Package:** `n8n-nodes-nvidia-nim` (latest: v2.3.4, Feb 2026)

**Steps:**
1. **Settings → Community Nodes → Install**
2. Enter: `n8n-nodes-nvidia-nim`
3. Accept risk → **Install**
4. Get API key from [build.nvidia.com](https://build.nvidia.com) → Sign up → API → Generate key
5. In n8n: create **NVIDIA NIM credential** → paste API key
6. Add **NVIDIA NIM Chat** node → select model → configure prompt

**Nodes added:** NVIDIA NIM Chat, NVIDIA NIM Image Analysis

#### Method 2 — HTTP Request Node

```
Method: POST
URL: https://integrate.api.nvidia.com/v1/chat/completions
Headers: Authorization: Bearer <YOUR_NVIDIA_API_KEY>
Body:
{
  "model": "meta/llama-3.1-70b-instruct",
  "messages": [{"role": "user", "content": "{{ $json.prompt }}"}],
  "max_tokens": 1024
}
Extract response: {{ $json.choices[0].message.content }}
```

#### Method 3 — OpenRouter (access NVIDIA models without NVIDIA account)

1. Sign up at [openrouter.ai](https://openrouter.ai)
2. Add OpenRouter Chat Model sub-node
3. Credential: OpenRouter API key
4. Model: `nvidia/llama-3.1-nemotron-70b-instruct`

#### Available NVIDIA NIM Models (examples)
```
meta/llama-3.1-70b-instruct
meta/llama-3.1-405b-instruct
nvidia/llama-3.1-nemotron-70b-instruct
nvidia/nemotron-4-340b-instruct
nvidia/llama-3.3-nemotron-super-49b-v1
meta/llama-3.2-11b-vision-instruct  (vision/multimodal)
mistralai/mistral-large-2-instruct
google/gemma-2-27b-it
microsoft/phi-3.5-mini-instruct
```

---

## Part 4 — Local LLM Integration (Step-by-Step)

### Why Run a Local LLM?

| Reason | Detail |
|--------|--------|
| **Privacy** | Data never leaves your machine or server |
| **Cost** | Zero API costs after hardware setup |
| **Offline** | Works without internet |
| **No rate limits** | Process as many tokens as your hardware allows |
| **Compliance** | HIPAA, GDPR, SOC 2 — no third-party data exposure |

---

### 🥇 Method 1 — Ollama (SIMPLEST — Start Here)

> Ollama is the recommended first choice. It is the simplest, most well-documented, and has the best n8n integration.

#### What is Ollama?

Ollama is an open-source tool (think "Docker for AI models") that:
- Downloads and runs LLMs locally with a single command
- Exposes an OpenAI-compatible REST API at `http://localhost:11434`
- Automatically manages GPU/CPU, quantization, and memory
- Supports 100+ models (Llama, Mistral, Phi, Gemma, DeepSeek, Qwen, etc.)

#### Hardware Requirements

| Model Size | Minimum RAM | Recommended GPU VRAM |
|------------|-------------|---------------------|
| 1–4B parameters | 4 GB RAM | No GPU needed |
| 7–8B parameters | 8 GB RAM | 6–8 GB VRAM |
| 13B parameters | 16 GB RAM | 12 GB VRAM |
| 30–34B parameters | 32 GB RAM | 24 GB VRAM |
| 70B parameters | 64 GB RAM | 48 GB VRAM |

> ℹ️ Quantized models (Q4, Q5, Q8) use less RAM. For example, Llama 3.1 8B Q4 runs comfortably on 8 GB RAM with no GPU.

---

#### Step 1 — Install Ollama

**Windows:**
1. Go to [https://ollama.com/download](https://ollama.com/download)
2. Click **Download for Windows**
3. Run the `.exe` installer
4. Ollama installs and starts automatically as a background service
5. Verify: Open Command Prompt → type `ollama --version` → should show version number

**macOS:**
```bash
# Option A — Download from website (easiest)
# Go to https://ollama.com/download → Download for macOS → open .dmg → drag to Applications

# Option B — Homebrew
brew install ollama
```

**Linux (Ubuntu/Debian):**
```bash
curl -fsSL https://ollama.com/install.sh | sh
```

**⚠️ Common install errors:**
- **Windows: "Ollama not found" in command prompt** → Restart the command prompt after install, or add `C:\Users\<you>\AppData\Local\Programs\Ollama` to your PATH
- **macOS: "app can't be opened" security warning** → System Preferences → Security & Privacy → Allow
- **Linux: Permission denied** → Run with `sudo` or add your user to the docker group if using Docker

---

#### Step 2 — Download a Model

```bash
# Lightweight model (3B) — good for testing on low-spec machines
ollama pull llama3.2:3b

# Balanced model (8B) — good for general use (needs 8GB RAM)
ollama pull llama3.1:8b

# Strong reasoning model (7B) — good balance of speed and quality
ollama pull mistral:7b

# Fast and efficient Microsoft model
ollama pull phi4:14b

# DeepSeek R1 (strong reasoning, 7B)
ollama pull deepseek-r1:7b

# List all installed models
ollama list

# Remove a model
ollama rm llama3.2:3b
```

**⚠️ Common model issues:**
- **Download stalls** → Check internet connection; large models (7B = ~4 GB). Use Ctrl+C and re-run `ollama pull` — it resumes from where it left off
- **"Error: model not found"** → Check exact model name at [ollama.com/library](https://ollama.com/library). Names are case-sensitive
- **Model list empty in n8n** → The model was downloaded but Ollama may need to be restarted: `ollama serve` in a new terminal

---

#### Step 3 — Verify Ollama is Running

```bash
# Test the Ollama API directly
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2:3b",
  "prompt": "Hello, are you running?",
  "stream": false
}'

# Expected response: {"model":"llama3.2:3b", "response":"Hello! Yes, I'm running...", ...}

# If no response:
ollama serve  # Start Ollama manually if it isn't running as a service
```

**⚠️ Common connection issues:**
- **`connection refused`** → Ollama is not running. Start it: `ollama serve` (Linux/Mac) or open the Ollama app (Windows/Mac)
- **Windows only** → Ollama runs in the system tray. Check the tray icon is visible

---

#### Step 4 — Connect Ollama to n8n

**Scenario A — n8n installed locally (same machine as Ollama)**

1. Open n8n in your browser (usually `http://localhost:5678`)
2. Create a new workflow
3. Add any AI Root node (e.g., **Basic LLM Chain**)
4. Click the **Chat Model** port → search for **Ollama Chat Model** → click it
5. Click **Select Credential** → **Create new credential**
6. **Base URL:** `http://localhost:11434` OR `http://127.0.0.1:11434`
7. Click **Test** → should show **"Connection tested successfully"**
8. Close credential window → in the node, open **Model** dropdown → select your model
9. If model dropdown is empty: refresh the browser and re-open the node

**⚠️ If connection fails on localhost:**
- Try `http://127.0.0.1:11434` instead of `http://localhost:11434` — some systems resolve differently
- Check Ollama is running: `curl http://localhost:11434` should return a response
- Check firewall: Windows Defender/Firewall may block port 11434 → allow it manually

---

**Scenario B — n8n in Docker, Ollama on host machine**

This is the most common setup for self-hosted n8n. Docker containers cannot use `localhost` to reach the host.

**Using Docker Desktop (Windows/Mac):**
```yaml
# In your docker-compose.yml or when running n8n:
# Use special Docker hostname: host.docker.internal
# In n8n Ollama credential Base URL: http://host.docker.internal:11434
```

**Using Docker on Linux (no Docker Desktop):**
```yaml
# host.docker.internal is NOT automatically available on Linux
# Add to your docker-compose.yml:
services:
  n8n:
    image: n8nio/n8n
    extra_hosts:
      - "host.docker.internal:host-gateway"
    ports:
      - "5678:5678"
```
Then use: `http://host.docker.internal:11434` in Ollama credential.

**Alternative — use host IP directly:**
```bash
# Find your host machine's IP (Linux)
ip route | grep default | awk '{print $3}'
# Example output: 172.17.0.1
# Use: http://172.17.0.1:11434
```

**⚠️ Docker issues:**
- **`host.docker.internal` not resolving on Linux** → Add `extra_hosts` to docker-compose as shown above
- **n8n can reach host IP but Ollama refuses connection** → By default Ollama only listens on `127.0.0.1`. Fix:
  ```bash
  # Set Ollama to listen on all interfaces (Linux/Mac)
  export OLLAMA_HOST=0.0.0.0
  ollama serve
  # Or add to systemd service: Environment="OLLAMA_HOST=0.0.0.0"
  ```

---

**Scenario C — n8n Cloud, Ollama on local machine (use ngrok)**

If you use n8n Cloud (cloud.n8n.io), your Ollama is behind your home/office router and n8n Cloud cannot reach `localhost`. Use ngrok to create a public HTTPS tunnel.

**Step-by-step with ngrok:**

1. **Install ngrok:**
   - Go to [ngrok.com](https://ngrok.com) → Sign up (free) → Download
   - Windows: run the `.exe`; Mac: `brew install ngrok`; Linux: `snap install ngrok`

2. **Authenticate ngrok:**
   ```bash
   ngrok config add-authtoken <YOUR_NGROK_TOKEN>
   # Token found at: dashboard.ngrok.com/get-started/your-authtoken
   ```

3. **Start the tunnel:**
   ```bash
   ngrok http 11434
   # Output shows something like:
   # Forwarding   https://abc123.ngrok-free.app -> http://localhost:11434
   ```

4. **Copy the HTTPS URL** (e.g., `https://abc123.ngrok-free.app`)

5. **In n8n Cloud Ollama credential:**
   - Base URL: `https://abc123.ngrok-free.app`
   - Click Test → should connect

6. **Select your model** in the Ollama Chat Model node

**⚠️ ngrok issues:**
- **Free tier generates a new URL every restart** → Restart ngrok → update the URL in n8n credential each time
- **ngrok warning page blocks requests** → Add `ngrok-skip-browser-warning: true` as a custom header in your Ollama credential
- **Persistent URL** → ngrok paid plans offer static domains; alternatively, self-host Cloudflare Tunnel (free) for a persistent URL
- **Ollama not reachable through ngrok** → Ensure `OLLAMA_HOST=0.0.0.0` is set so Ollama listens on all interfaces, not just localhost

---

**Scenario D — Ollama on a separate server/NAS (e.g., QNAP)**

```bash
# Install Ollama on your QNAP/server via Container Station (Docker)
docker run -d --name ollama \
  -p 11434:11434 \
  -v /share/ollama:/root/.ollama \
  -e OLLAMA_HOST=0.0.0.0 \
  ollama/ollama

# Pull a model on your NAS/server:
docker exec -it ollama ollama pull llama3.1:8b

# In n8n Ollama credential Base URL:
http://<NAS_IP_ADDRESS>:11434
# Example: http://192.168.1.100:11434
```

**⚠️ Server/NAS issues:**
- **Connection timeout** → Check your router/firewall allows port 11434 between devices on the LAN
- **Model runs slowly on NAS CPU** → Use a smaller quantized model (e.g., `llama3.2:3b` instead of `llama3.1:8b`)

---

#### Step 5 — Build Your First Local AI Workflow

**Simple chatbot with local Ollama:**

```
Chat Trigger
    └── Basic LLM Chain
          └── [Chat Model] Ollama Chat Model
                          Credential: Ollama (http://localhost:11434)
                          Model: llama3.1:8b
```

**Full AI Agent with local LLM:**

```
Chat Trigger
    └── AI Agent
          ├── [Chat Model] Ollama Chat Model (llama3.1:8b)
          ├── [Memory]     Simple Memory
          └── [Tools]      Calculator + Wikipedia + HTTP Request Tool
```

**System prompt example:**
```
You are an IT support assistant. You help users troubleshoot common IT issues.
Always ask clarifying questions before suggesting a solution.
Be concise and professional.
Today is {{ $now.toFormat('dd MMMM yyyy') }}.
```

---

#### Step 6 — Troubleshooting Ollama in n8n

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Model dropdown is empty | Ollama has no models installed | Run `ollama pull llama3.1:8b` in terminal |
| "Connection tested successfully" but model still empty | Browser cache | Refresh n8n browser tab, reopen node |
| Slow responses | Model too large for hardware | Switch to smaller model (3B or 7B Q4) |
| n8n workflow times out | Model loading time too long | Increase HTTP timeout in n8n Settings, or pre-load model: `ollama run llama3.1:8b` before workflow |
| "context length exceeded" error | Input too long for model | Use Text Splitter + Summarization Chain; reduce context window |
| Docker: "connection refused" | Ollama not accessible from container | Use `host.docker.internal:11434` and add `extra_hosts` in Linux |
| Model responses are garbled | Wrong model format | Try a different model; some models require specific prompt formats |
| Ollama crashes under load | Insufficient RAM | Use smaller model; add swap space; reduce parallel requests (`OLLAMA_NUM_PARALLEL=1`) |

---

#### Step 7 — Recommended Models by Use Case

| Use Case | Recommended Model | Command | Notes |
|----------|------------------|---------|-------|
| General chat | llama3.1:8b | `ollama pull llama3.1:8b` | Best all-rounder |
| Low-spec machine | llama3.2:3b | `ollama pull llama3.2:3b` | Runs on 4GB RAM |
| Fast responses | phi4:14b | `ollama pull phi4:14b` | Fast + smart |
| Code assistance | deepseek-r1:7b | `ollama pull deepseek-r1:7b` | Strong reasoning |
| Long context | qwen2.5:7b | `ollama pull qwen2.5:7b` | 128K context |
| Multimodal | llava:7b | `ollama pull llava:7b` | Vision + text |
| Embeddings | nomic-embed-text | `ollama pull nomic-embed-text` | For RAG pipelines |

---

### 🥈 Method 2 — LM Studio (GUI Alternative)

**Best for:** Non-developers, those who prefer a visual interface.

| Attribute | Detail |
|-----------|--------|
| **What it is** | A desktop app for running LLMs locally with a chat GUI |
| **Platform** | Windows, macOS, Linux |
| **API** | Exposes OpenAI-compatible API on port 1234 by default |
| **Models** | Download models from Hugging Face inside the app |

**Steps:**
1. Download from [lmstudio.ai](https://lmstudio.ai)
2. Install and open LM Studio
3. Go to the **Discover** tab → search for a model → download
4. Go to **Local Server** tab → select your model → click **Start Server**
5. Note the server URL (default: `http://localhost:1234/v1`)
6. In n8n: add **OpenAI Chat Model** sub-node
7. Credential: Use a custom OpenAI credential with:
   - API Key: `lm-studio` (any string works)
   - Base URL: `http://localhost:1234/v1`
8. Model: type the exact model name shown in LM Studio

**⚠️ LM Studio issues:**
- **n8n can't find model** → LM Studio must have a model loaded AND the server must be started
- **Docker n8n → LM Studio** → Same `host.docker.internal` solution as Ollama
- **Slow startup** → LM Studio loads the model on first request; pre-load by sending a test message in the LM Studio chat

---

### 🥉 Method 3 — GPT4All (Simplest GUI)

**Best for:** Complete beginners — all-in-one, no CLI needed.

| Attribute | Detail |
|-----------|--------|
| **What it is** | Desktop app with built-in model library and local server |
| **Platform** | Windows, macOS, Linux |
| **API** | OpenAI-compatible on port 4891 |

**Steps:**
1. Download from [gpt4all.io](https://gpt4all.io)
2. Install → open → download a model from the Models tab
3. Go to Settings → Enable API server
4. In n8n: OpenAI Chat Model credential:
   - API Key: `not-needed`
   - Base URL: `http://localhost:4891/v1`

---

## Part 5 — Sub-nodes Reference

### Embeddings Sub-nodes

| Node | Provider | Free? | Best For |
|------|---------|-------|----------|
| Embeddings Ollama | Local | ✅ Free | Local RAG |
| Embeddings HuggingFace Inference | HuggingFace | ✅ Free tier | Research/dev |
| Embeddings OpenAI | OpenAI | ❌ Paid | Production |
| Embeddings Google Gemini | Google | ✅ Free tier | Google stack |
| Embeddings Mistral Cloud | Mistral | ✅ Free tier | EU-compliant |
| Embeddings Cohere | Cohere | ✅ Free tier | RAG |
| Embeddings Azure OpenAI | Azure | ❌ Paid | Azure stack |
| Embeddings AWS Bedrock | AWS | Limited | AWS stack |
| Embeddings Google PaLM | Google | ⚠️ Deprecated | Avoid new use |

**Local embeddings with Ollama (for RAG):**
```
# Download an embedding model
ollama pull nomic-embed-text

# In n8n: Embeddings Ollama sub-node
# Credential: same Ollama credential
# Model: nomic-embed-text
```

---

### Memory Sub-nodes

| Node | Persistence | Use Case |
|------|-------------|----------|
| **Simple Memory** | Session only ⚠️ | Testing |
| **Redis Chat Memory** | Persistent | Scalable production |
| **Postgres Chat Memory** | Persistent | SQL stack |
| **MongoDB Chat Memory** | Persistent | Document stack |
| **Motorhead** | External | Persistent memory backend |
| **Xata** | Cloud | Cloud DB memory |
| **Zep** | External | Long-term memory + vectors |

**Session ID tip:** Use `{{ $sessionId }}` to automatically group messages by conversation.

---

### Output Parser Sub-nodes

| Node | Purpose | When to Use | ⚠️ Notes |
|------|---------|------------|----------|
| **Structured Output Parser** | Enforce JSON schema on LLM output | When you need consistent JSON | Fails if LLM doesn't comply ⚠️ |
| **Auto-fixing Output Parser** | Retry + fix LLM formatting errors | When LLM output format is inconsistent | Not 100% reliable ⚠️ |
| **Item List Output Parser** | Parse a list from LLM response | When expecting bullet list / numbered list | Needs consistent LLM output |

---

## Part 6 — Complete AI Workflow Examples

### Example 1 — Local IT Helpdesk Agent (Ollama + Tools)

```
Chat Trigger
    └── AI Agent
          ├── [Model]   Ollama Chat Model (llama3.1:8b)
          ├── [Memory]  Postgres Chat Memory (session: {{ $sessionId }})
          └── [Tools]
                ├── HTTP Request Tool → ServiceNow API
                ├── HTTP Request Tool → JIRA API
                └── Call Workflow Tool → "Send Email Notification" workflow
```

**System prompt:**
```
You are an IT helpdesk assistant.
You can look up assets in ServiceNow and create tickets in JIRA.
Always confirm before creating or modifying any record.
Current date: {{ $now.toFormat('dd MMM yyyy') }}
```

---

### Example 2 — RAG Pipeline with Local Models

**Build phase (index documents):**
```
Manual Trigger
    → Read Files
    → Default Data Loader
    → Recursive Character Text Splitter
    → PGVector Vector Store [Insert mode]
          └── Embeddings Ollama (nomic-embed-text)
```

**Query phase:**
```
Webhook
    → Q&A Chain
          ├── [Model]     Ollama Chat Model (llama3.1:8b)
          └── [Retriever] Vector Store Retriever
                └── PGVector Vector Store [Retrieve mode]
                      └── Embeddings Ollama (nomic-embed-text)
    → Respond to Webhook
```

---

### Example 3 — Multi-Model Routing (Groq + Ollama fallback)

```
Chat Trigger
    → If ({{ $json.chatInput.length }} > 500 characters)
          [True]  → Basic LLM Chain → Ollama (local, large context)
          [False] → Basic LLM Chain → Groq (fast cloud, small prompts)
    → Merge
    → Respond to Webhook
```

---

## Part 7 — Decision Guide: Which LLM to Use?

```
Do you have privacy/compliance requirements?
│
├── YES → Use Ollama (local) or LM Studio
│          Choose model based on hardware:
│          < 8GB RAM → llama3.2:3b
│          8–16GB RAM → llama3.1:8b or mistral:7b
│          16GB+ RAM → phi4:14b or qwen2.5:14b
│
└── NO → Use cloud API
         │
         ├── Need speed? → Groq (fastest)
         ├── Need free?  → Groq or Google Gemini free tier
         ├── Need power? → OpenAI GPT-4o or Anthropic claude-sonnet
         ├── Need low cost? → DeepSeek (cheapest capable model)
         ├── Need 100+ models from one API? → OpenRouter
         └── Need NVIDIA GPU models? → NVIDIA NIM (build.nvidia.com)
```

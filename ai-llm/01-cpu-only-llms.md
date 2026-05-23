# CPU-Only LLMs — Run AI Without a GPU

> **Author:** Shrinivas Ramaprasad | **Updated:** May 2026
> **Reference:** https://localaimaster.com/blog/ollama-system-requirements

---

## Why This Matters for n8n

Most guides assume you have an NVIDIA GPU. If you're running n8n on a laptop, a NAS, a budget VPS, or a Mac — you may have no dedicated GPU at all. The good news: **you can run capable LLMs on CPU only**. This guide tells you exactly which models work, how fast they'll be, how to configure them in n8n, and what to do when context limits hit.

---

## The Key Facts

- **CPU inference works.** Ollama runs on CPU with no GPU present.
- **Speed is slower.** Expect 5–15 tokens/second on a modern CPU vs 40–80+ on a GPU.
- **RAM is the constraint**, not VRAM. You need ~0.6 GB RAM per billion parameters at Q4 quantization.
- **Quantization is your friend.** A 7B model at Q4 (4-bit) uses ~4.5 GB RAM instead of 14 GB at FP16.
- **Apple Silicon (M1/M2/M3/M4) is the exception** — unified memory means these chips run LLMs almost as fast as a mid-range GPU.

---

## Hardware Requirements by Model Size

| Model Size | Min RAM (CPU) | Comfortable RAM | Tokens/sec (CPU) | Notes |
|---|---|---|---|---|
| **0.5B–1B params** | 2 GB | 4 GB | 20–40 t/s | Very fast on CPU. Limited capability. |
| **3B–4B params** | 4 GB | 6 GB | 10–20 t/s | Sweet spot for CPU-only. Good quality. |
| **7B–8B params** | 8 GB | 12 GB | 5–12 t/s | Usable. Slow for long responses. |
| **13B–14B params** | 12 GB | 16 GB | 3–6 t/s | Slow on most CPUs. |
| **32B+ params** | 24 GB | 32 GB | 1–3 t/s | Painful on CPU. Not recommended. |

> **Rule of thumb:** If you have 16 GB RAM → run up to 8B models. 8 GB RAM → stick to 3B–4B models.

---

## Recommended Models for CPU-Only Use

### Tier 1: Best for 4–6 GB RAM (Ultra-light, fast)

| Model | Size | Pull Command | Best For |
|---|---|---|---|
| **Qwen3:0.6B** | 0.6B, ~0.5 GB | `ollama pull qwen3:0.6b` | Ultra-fast, simple tasks |
| **Gemma3:1B** | 1B, ~0.8 GB | `ollama pull gemma3:1b` | Light chatbot, simple classification |
| **TinyLlama 1.1B** | 1.1B, ~0.6 GB | `ollama pull tinyllama` | Legacy, basic use |

### Tier 2: Best for 6–10 GB RAM (Recommended balance)

| Model | Size | Pull Command | Best For | Context Window |
|---|---|---|---|---|
| **Phi-3 Mini (Q4)** | 3.8B, ~2.2 GB | `ollama pull phi3:mini` | **Best CPU model overall** — strong reasoning | 4K tokens |
| **Phi-4 Mini (Q4)** | 3.8B, ~2.5 GB | `ollama pull phi4-mini` | Improved Phi-3, better instruction following | 16K tokens |
| **Llama3.2:3B (Q4)** | 3B, ~1.9 GB | `ollama pull llama3.2:3b` | General purpose, good quality | 128K tokens |
| **Qwen2.5:3B (Q4)** | 3B, ~1.9 GB | `ollama pull qwen2.5:3b` | Good multilingual support | 32K tokens |
| **Gemma3:4B (Q4)** | 4B, ~2.5 GB | `ollama pull gemma3:4b` | Google's model, good instruction following | 128K tokens |

### Tier 3: For 10–16 GB RAM (Good quality, slower)

| Model | Size | Pull Command | Best For |
|---|---|---|---|
| **Llama3.1:8B (Q4)** | 8B, ~4.7 GB | `ollama pull llama3.1:8b` | Best general purpose in 8B class |
| **Mistral:7B (Q4)** | 7B, ~4.1 GB | `ollama pull mistral:7b` | Long context tasks (32K) |
| **DeepSeek-R1:7B (Q4)** | 7B, ~4.7 GB | `ollama pull deepseek-r1:7b` | Reasoning, step-by-step thinking |
| **Qwen2.5:7B (Q4)** | 7B, ~4.7 GB | `ollama pull qwen2.5:7b` | Long context (128K), multilingual |

---

## Performance Benchmarks (CPU-Only)

| Hardware | Model | Tokens/sec |
|---|---|---|
| Intel i7-1185G7, 16 GB RAM (no GPU) | Llama3.2:3B Q4 | ~8 t/s |
| Intel i7-1185G7, 16 GB RAM (no GPU) | Phi-3 Mini Q4 | ~10 t/s |
| Intel i5 8th Gen, 16 GB RAM | Phi-3 Mini Q4 | ~6 t/s |
| AMD Ryzen 7 5800X, 32 GB RAM | Llama3.1:8B Q4 | ~9 t/s |
| Apple M2, 16 GB unified | Llama3.1:8B Q4 | ~35 t/s |
| Apple M3, 16 GB unified | Llama3.1:8B Q4 | ~45 t/s |
| QNAP NAS (quad-core ARM) | Llama3.2:3B Q4 | ~2–4 t/s |

> **Apple Silicon note:** M-series Macs use unified memory — the GPU and CPU share the same RAM pool. This means even without a discrete GPU, inference speed is dramatically faster.

---

## Optimise CPU Inference with Ollama

```bash
# Set thread count (match your CPU core count)
export OLLAMA_NUM_THREADS=8
ollama serve

# Force CPU-only even if GPU is present
export CUDA_VISIBLE_DEVICES=""
ollama run llama3.2:3b

# Check if AVX-512 is supported (10-20% faster inference)
grep avx512 /proc/cpuinfo | head -1
# Intel 12th Gen+ and AMD Zen 4+ support AVX-512

# Close RAM-heavy apps before running
# Browsers, IDEs, etc. compete for the same RAM Ollama needs
```

---

## How to Configure CPU-Only Models in n8n

### Step 1 — Install Ollama

```bash
# macOS / Linux
curl -fsSL https://ollama.com/install.sh | sh

# Windows: download from https://ollama.com/download
```

### Step 2 — Pull a CPU-friendly model

```bash
# For 8–16 GB RAM laptop (recommended starting point)
ollama pull phi3:mini

# Verify it works
ollama run phi3:mini "Hello, are you running on CPU?"
```

### Step 3 — Add Ollama Chat Model in n8n

```
1. In n8n workflow: add AI Agent (or Basic LLM Chain)
2. Click Chat Model port → add Ollama Chat Model sub-node
3. Click Credential → Create new
   Base URL: http://localhost:11434
   (if n8n is in Docker: http://host.docker.internal:11434)
4. Test → ✅ Connected
5. Model dropdown → select phi3:mini (or your pulled model)
6. Options (optional):
   Temperature: 0.3–0.5 (lower = more consistent)
   Context Window: 4096 (default) — reduce to 2048 to save RAM
```

### Step 4 — Docker networking (if n8n is in Docker)

```
Scenario A — Docker Desktop (Windows/Mac):
  Base URL: http://host.docker.internal:11434

Scenario B — Linux Docker:
  Add to docker-compose.yml n8n service:
  extra_hosts:
    - "host.docker.internal:host-gateway"
  Then use: http://host.docker.internal:11434

Scenario C — Both n8n and Ollama in Docker Compose:
  Base URL: http://ollama:11434
```

---

## Context Length Limits — The Problem and Solutions

### The Problem

Every LLM has a maximum context window (input + output combined). CPU models tend to be smaller and have shorter context windows. When your workflow sends too much text, the model either truncates it or throws an error.

| Model | Context Window | Max Input (practical) |
|---|---|---|
| Phi-3 Mini | 4,096 tokens | ~2,500 words input |
| Llama3.2:3B | 128,000 tokens | Very long |
| Gemma3:1B | 8,192 tokens | ~5,000 words |
| Mistral:7B | 32,768 tokens | ~20,000 words |
| Phi-4 Mini | 16,384 tokens | ~10,000 words |

> ~1 token ≈ 0.75 words in English.

### Solution 1: Text Splitter + Map Reduce (Best for Documents)

```
Input Document (long)
    ↓
Recursive Character Text Splitter
    Chunk size: 1000 tokens
    Overlap: 100 tokens
    ↓
Loop Over Items
    ↓
  Basic LLM Chain (summarise/extract each chunk)
  Model: phi3:mini (fits in 4K context)
    ↓
Merge all chunk results
    ↓
Basic LLM Chain (final summary of summaries)
```

### Solution 2: Multi-Model Fallback Chain

```
Input Text
    ↓
Code Node: check token count
  if (text.length < 2000) → use phi3:mini (fast, CPU)
  else if (text.length < 8000) → use llama3.2:3b (slightly slower)
  else → use groq API (free cloud, no context limit problem)
    ↓
If Node: which model?
    ├── Short → Ollama phi3:mini
    ├── Medium → Ollama llama3.2:3b
    └── Long → Groq Cloud API (backup)
```

### Solution 3: Summarise Before Processing

```
Long Input
    ↓
Summarization Chain
    Model: llama3.2:3b (128K context)
    Mode: Map Reduce
    ↓
Short Summary (~500 words)
    ↓
Main Processing Chain
    Model: phi3:mini (fine for 500 words)
```

### Solution 4: Context Trimming in Code Node

```javascript
// Code Node — trim input to fit model context
const text = $json.inputText;
const MAX_CHARS = 3000; // ~2000 tokens for phi3:mini

const trimmed = text.length > MAX_CHARS
  ? text.substring(0, MAX_CHARS) + '... [truncated]'
  : text;

return [{ json: { text: trimmed, was_truncated: text.length > MAX_CHARS } }];
```

---

## Multi-Provider Fallback Strategy for n8n

When a free API provider hits its rate limit or context limit, automatically fall back to the next one:

```
Input
  ↓
If: text < 3000 chars AND groq_calls_today < 14000
  ↓ YES
  Groq API (fastest, free)
  ↓ NO
If: text < 6000 chars
  ↓ YES
  Ollama Phi-3 Mini (local, free)
  ↓ NO
If: text < 16000 chars
  ↓ YES
  Ollama Llama3.2:3B (local, 128K context)
  ↓ NO
Google Gemini API (free tier, 1M+ context)
```

### Rate limits reference for fallback logic

| Provider | Daily Limit | RPM | Context |
|---|---|---|---|
| **Groq** (free) | 14,400 req | 30 | 32K–128K |
| **OpenRouter** free models | 50 req/day | 20 | varies |
| **Google Gemini** (free) | 1,500 req | 15 | 1M |
| **Mistral** (free) | 500K tokens/month | 1 RPS | 128K |
| **Ollama** (local) | Unlimited | Hardware | Model-dependent |

---

## LLM Tools — No GPU Required

| Tool | What it is | Platform | URL |
|---|---|---|---|
| **Ollama** | CLI runner, REST API server | Win/Mac/Linux | https://ollama.com |
| **LM Studio** | GUI with built-in chat + API server | Win/Mac/Linux | https://lmstudio.ai |
| **GPT4All** | One-click GUI, model browser | Win/Mac/Linux | https://gpt4all.io |
| **Jan** | Open-source ChatGPT-style desktop app | Win/Mac/Linux | https://jan.ai |
| **Open WebUI** | Browser-based chat UI for Ollama | Any (Docker) | https://openwebui.com |

**Recommendation:** Ollama for n8n integration (REST API, easiest). Open WebUI on top of Ollama for a ChatGPT-style UI.

---

## Connect Ollama to n8n via Tailscale (VPS Architecture)

When n8n runs on a remote VPS and Ollama runs on your local laptop, use Tailscale as the secure private tunnel. This avoids exposing Ollama to the public internet — Ollama has no built-in authentication, so a public tunnel like ngrok makes your LLM API reachable by anyone.

**Interactive Flowchart:** [ollama-tailscale-n8n-flowchart.html](./ollama-tailscale-n8n-flowchart.html)

Covers: architecture overview, full setup steps (Tailscale, Ollama Windows config, n8n credential), and a 5-check diagnostic flow for when something breaks.

| Component | Location | Address |
|---|---|---|
| n8n | Hostinger VPS (srv1642799) | 100.110.9.89 |
| Tailscale | Secure private mesh | No public ports |
| Ollama | Windows Laptop | 100.94.10.77:11434 |
| Models | Local — zero API cost | mistral 7B · gemma4 8B · phi4-mini 3.8B · qwen3 4B |

**Critical Windows fixes:**

- `OLLAMA_HOST` must be `0.0.0.0`. Default is `127.0.0.1` (localhost only).
  - Permanent: `setx OLLAMA_HOST "0.0.0.0" /M` (Admin CMD) then restart Ollama.
  - Current session only: `set OLLAMA_HOST=0.0.0.0` then `ollama serve`.
- Windows Firewall needs an inbound rule for TCP port 11434 (all 3 profiles via `wf.msc`).
- Tailscale zombie state: if ping times out despite showing Connected, run `Restart-Service Tailscale` in Admin PowerShell.

**n8n credential settings:**

```
Credential type: Ollama
Name: Ollama Local (Tailscale)
Base URL: http://100.94.10.77:11434
API Key: (leave blank)
```

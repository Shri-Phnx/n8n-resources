# Open-Source & Free LLM API Limits — Complete Reference

> **Author:** Shrinivas Ramaprasad  
> **Last updated:** April 2026  
> **⚠️ Limits change frequently — always verify at each provider's official dashboard before building production workflows.**

---

## Why Limits Matter in n8n

When you build n8n workflows with free/open-source LLM APIs, hitting a rate limit means:
- The node throws a `429 Too Many Requests` error
- The workflow execution fails mid-way
- Data may be partially processed
- Downstream nodes never execute

**Solution:** Use the **Wait node** (1–5 seconds) between LLM calls, enable **Retry On Fail** in the node Settings tab, and monitor your quota on the provider dashboard.

---

## 1. Groq — Free Tier Limits

**Dashboard:** [console.groq.com](https://console.groq.com)

### Rate Limits by Model (Free Tier, April 2026)

| Model | Requests/Min | Tokens/Min | Requests/Day | Tokens/Day |
|-------|-------------|-----------|-------------|----------|
| `llama-3.3-70b-versatile` | 30 | 6,000 | 14,400 | 500,000 |
| `llama-3.1-8b-instant` | 30 | 20,000 | 14,400 | 500,000 |
| `llama-3.2-11b-vision-preview` | 15 | 7,000 | 7,000 | 250,000 |
| `llama-3.2-90b-vision-preview` | 15 | 7,000 | 3,500 | 250,000 |
| `mixtral-8x7b-32768` | 30 | 5,000 | 14,400 | 500,000 |
| `gemma2-9b-it` | 30 | 15,000 | 14,400 | 500,000 |
| `deepseek-r1-distill-llama-70b` | 30 | 6,000 | 14,400 | 500,000 |

### Model Limitations

| Model | Strengths | Weaknesses |
|-------|----------|------------|
| `llama-3.3-70b-versatile` | Best quality on free tier, strong reasoning | Slower than 8B, lower token/min limit |
| `llama-3.1-8b-instant` | Fastest response time, highest token/min | Less capable for complex reasoning |
| `mixtral-8x7b-32768` | 32K context window | Older architecture, lower quality than Llama 3.3 |
| `gemma2-9b-it` | Good for structured output | Less capable for long-form content |
| `deepseek-r1-distill-llama-70b` | Strong reasoning/math | Slower, verbose chain-of-thought |

### n8n Handling:
```
Groq Chat Model → [429 error]?
Fix: Add Wait node (5 seconds) before Groq node.
Alternative: Enable "Retry On Fail" → 3 retries, 5000ms wait.
```

---

## 2. OpenRouter — Free Model Limits

**Dashboard:** [openrouter.ai/activity](https://openrouter.ai/activity)

### Free Model Limits (models ending in `:free`)

| Model | Requests/Min | Context Window | Daily Limit |
|-------|-------------|---------------|-------------|
| `meta-llama/llama-3.3-70b-instruct:free` | 20 | 131,072 tokens | 50 req/day per IP |
| `google/gemma-2-9b-it:free` | 20 | 8,192 tokens | 50 req/day per IP |
| `mistralai/mistral-7b-instruct:free` | 20 | 32,768 tokens | 50 req/day per IP |
| `deepseek/deepseek-r1:free` | 20 | 163,840 tokens | 50 req/day per IP |
| `meta-llama/llama-3.1-8b-instruct:free` | 20 | 131,072 tokens | 50 req/day per IP |

### Important Notes for OpenRouter

| Note | Detail |
|------|--------|
| **IP-based limits on free models** | Free models are limited per IP address — self-hosted n8n on a shared VPS may share limits with others |
| **Rate limits vary by model** | Paid models have much higher limits. Check the model's info page on openrouter.ai |
| **Credit-based system** | You can add credits ($5 minimum) to unlock higher limits on all models |
| **Model routing** | OpenRouter can automatically route to an available model if your first choice is throttled — configure in their settings |

### Model Quality Comparison

| Model | Best For | Avoid For |
|-------|---------|----------|
| `llama-3.3-70b:free` | General reasoning, instructions | Very long documents (slow) |
| `deepseek-r1:free` | Math, code, structured analysis | Casual conversation (overly verbose) |
| `mistral-7b:free` | Fast tasks, classification | Complex multi-step reasoning |
| `gemma-2-9b:free` | Short tasks, summarisation | Long context (8K limit) |

---

## 3. NVIDIA NIM — Free Tier Limits

**Dashboard:** [build.nvidia.com](https://build.nvidia.com)

### Free API Credits (API Catalog)

| Tier | Credits | When to Use |
|------|---------|-------------|
| **Free / Trial** | ~1,000 API calls on signup | Testing and development only |
| **NIM Day 0** | Free for exploration | New models, early access |
| **NIM Turbo** | Free in production | Performance-critical apps |
| **NIM Certified** | Requires NVIDIA AI Enterprise license ($) | Enterprise regulated environments |

### Model Limitations

| Model | Context | Max Tokens Out | Notes |
|-------|---------|---------------|-------|
| `meta/llama-3.1-70b-instruct` | 128K | 4,096 | Strong all-rounder |
| `meta/llama-3.1-405b-instruct` | 128K | 4,096 | Highest quality, slower |
| `nvidia/nemotron-4-340b-instruct` | 4K | 4,096 | Best for NVIDIA use cases |
| `meta/llama-3.2-11b-vision-instruct` | 128K | 4,096 | Vision/multimodal |
| `microsoft/phi-3.5-mini-instruct` | 128K | 4,096 | Small, fast |

### n8n HTTP Request error handling for NIM:
```json
// 429 response body:
{ "error": { "message": "Rate limit exceeded", "type": "requests" } }

// Fix: Enable Retry On Fail in Settings tab
// Max tries: 3, Wait between: 5000ms
```

---

## 4. Hugging Face Inference API — Free Tier

**Dashboard:** [huggingface.co/settings/billing](https://huggingface.co/settings/billing)

### Free Tier Limits

| Feature | Free Tier | Pro ($9/month) |
|---------|----------|----------------|
| Serverless Inference calls | Rate-limited | Higher limits |
| Concurrent requests | 1 | Multiple |
| Model load wait | Up to 60 seconds | Instant (warm) |
| Private model inference | ❌ | ✅ |

### Model-Specific Notes

| Model | Size | Free? | Context | Limitation |
|-------|------|-------|---------|------------|
| `microsoft/Phi-3-mini-4k-instruct` | 3.8B | ✅ | 4K | Small context window |
| `mistralai/Mistral-7B-Instruct-v0.3` | 7B | ✅ | 32K | May be cold/slow |
| `google/gemma-7b-it` | 7B | ✅ | 8K | Rate limited |
| `meta-llama/Meta-Llama-3-8B-Instruct` | 8B | ✅ | 8K | Requires accepting model terms on HF |
| `HuggingFaceH4/zephyr-7b-beta` | 7B | ✅ | 32K | Good quality free option |

### Common Errors in n8n

| Error | Cause | Fix |
|-------|-------|-----|
| `Model is currently loading` | Cold start — model not warmed up | Add **Wait** node (30 seconds) before HF node, enable **Retry On Fail** |
| `503 Service Unavailable` | Model overloaded or down | Retry with backoff; switch to a different model |
| `Token limit exceeded` | Input too long | Use Text Splitter node before HF node |
| `Unauthorized 401` | Token expired or wrong scopes | Refresh token in n8n Credentials |
| `You need to agree to the model terms` | Model requires explicit acceptance | Visit huggingface.co/model-name and accept terms while logged in |

---

## 5. Mistral AI — Free Tier Limits

**Dashboard:** [console.mistral.ai](https://console.mistral.ai)

### Free Tier (Experiment/Trial)

| Limit | Value |
|-------|-------|
| Requests per second | 1 |
| Requests per month | 500,000 tokens total |
| Context window | Up to model limit |

### Model Comparison

| Model | Context | Best For | Notes |
|-------|---------|---------|-------|
| `mistral-large-latest` | 128K | Best quality reasoning | Most tokens — slower |
| `mistral-small-latest` | 128K | Fast, cost-effective | Good quality/speed balance |
| `mistral-nemo` | 128K | Lightweight | 12B — very fast |
| `codestral-latest` | 256K | Code generation | Best context for code |
| `open-mistral-nemo` | 128K | Open-weight local use | Same as mistral-nemo but free |

### EU Data Residency Note

> Mistral AI is a French company. All inference is processed within the EU. This makes it the best choice for GDPR-sensitive workflows.

---

## 6. Cohere — Free Tier Limits

**Dashboard:** [dashboard.cohere.com](https://dashboard.cohere.com)

### Trial API Key Limits

| Feature | Limit |
|---------|-------|
| Requests per minute | 5 |
| Requests per month | 1,000 |
| Generate tokens per call | 4,096 |
| Embed batch size | 96 items |

### Model Comparison

| Model | Context | Best For |
|-------|---------|----------|
| `command-r-plus` | 128K | Best quality, RAG, tool use |
| `command-r` | 128K | Good quality, faster |
| `command-light` | 4K | Fastest, cheapest |
| `embed-english-v3.0` | N/A | English embeddings for RAG |
| `rerank-english-v3.0` | N/A | Rerank RAG search results |

---

## 7. DeepSeek — Pricing (No Free Tier)

**Dashboard:** [platform.deepseek.com](https://platform.deepseek.com)

### Pricing (as of April 2026)

| Model | Input (per 1M tokens) | Output (per 1M tokens) |
|-------|----------------------|----------------------|
| `deepseek-chat` (V3) | $0.27 | $1.10 |
| `deepseek-reasoner` (R1) | $0.55 | $2.19 |

### Rate Limits

| Plan | RPM | TPM |
|------|-----|-----|
| Default | 60 | 60,000 |
| Higher tier | Contact sales | Contact sales |

### Model Notes

| Model | Context | Best For | Limitation |
|-------|---------|---------|------------|
| `deepseek-chat` (V3) | 64K | Coding, analysis, chat | No free tier |
| `deepseek-reasoner` (R1) | 64K | Chain-of-thought, math, logic | Verbose — outputs thinking trace which increases token cost |

---

## 8. xAI Grok — Limits

**Dashboard:** [console.x.ai](https://console.x.ai)

### Free Tier (as of April 2026)

| Note | Detail |
|------|--------|
| **Availability** | Limited free credits on signup — check current offer at console.x.ai |
| **Access** | API access requires a valid X (Twitter) account |

### Model Notes

| Model | Context | Best For |
|-------|---------|----------|
| `grok-3` | 131K | Most capable — reasoning, complex tasks |
| `grok-2` | 131K | Faster, lower cost |
| `grok-2-vision` | 8K | Image + text input |

---

## 9. Google Gemini — Free Tier Limits

**Dashboard:** [aistudio.google.com](https://aistudio.google.com)

### Gemini API Free Tier (Google AI Studio)

| Model | Requests/Min | Requests/Day | Tokens/Min |
|-------|-------------|-------------|----------|
| `gemini-2.0-flash` | 15 | 1,500 | 1,000,000 |
| `gemini-1.5-flash` | 15 | 1,500 | 1,000,000 |
| `gemini-1.5-pro` | 2 | 50 | 32,000 |

### Model Notes

| Model | Context | Best For | Limitation |
|-------|---------|---------|------------|
| `gemini-2.0-flash` | 1M tokens | Multimodal, fast tasks | Rate limited on free tier |
| `gemini-1.5-pro` | 2M tokens | Very long documents, RAG | Only 2 RPM on free tier — ⚠️ easily exceeded |

---

## 10. Ollama (Local) — Hardware Limits (Not API Limits)

Ollama has no API rate limits — you are limited only by your hardware.

### Performance by Hardware

| Hardware | Model | Tokens/Second | Notes |
|---------|-------|--------------|-------|
| MacBook Air M2 (8GB RAM) | llama3.2:3b | ~30 t/s | Good for development |
| MacBook Pro M3 (16GB RAM) | llama3.1:8b | ~20 t/s | Good quality/speed balance |
| MacBook Pro M3 Max (64GB RAM) | llama3.1:70b | ~15 t/s | Near-cloud quality |
| Windows PC (RTX 3090, 24GB VRAM) | llama3.1:8b | ~80 t/s | Excellent speed |
| Windows PC (RTX 4090, 24GB VRAM) | llama3.1:8b | ~120 t/s | Best consumer GPU |
| QNAP NAS (CPU only) | llama3.2:3b | ~5 t/s | Usable for low-traffic |
| Budget VPS (2 CPU, 4GB RAM) | llama3.2:3b (quantized) | ~2 t/s | Very slow — only for testing |

### Model Memory Requirements

| Model | Disk | RAM/VRAM Needed |
|-------|------|----------------|
| `phi3:mini` (3.8B Q4) | 2.2 GB | 4 GB |
| `llama3.2:3b` (Q4) | 1.9 GB | 4 GB |
| `mistral:7b` (Q4) | 4.1 GB | 6 GB |
| `llama3.1:8b` (Q4) | 4.7 GB | 8 GB |
| `phi4:14b` (Q4) | 9.1 GB | 12 GB |
| `qwen2.5:32b` (Q4) | 20 GB | 24 GB |
| `llama3.1:70b` (Q4) | 40 GB | 48 GB |

### Concurrent Request Limits for Ollama

| Setting | Default | How to Change |
|---------|---------|---------------|
| Parallel requests | 1 | `OLLAMA_NUM_PARALLEL=4` (uses more VRAM) |
| Max loaded models | 1 | `OLLAMA_MAX_LOADED_MODELS=2` |
| Context length | Model default | Set per-request via `num_ctx` parameter |

---

## Quick Reference — Free Tier Comparison

| Provider | Daily Limit | RPM | Best Free Model | Best For |
|----------|------------|-----|----------------|----------|
| **Groq** | 14,400 req | 30 | llama-3.3-70b | Volume automation |
| **OpenRouter** | 50 req (free models) | 20 | llama-3.3-70b:free | Variety, testing |
| **Google Gemini** | 1,500 req | 15 | gemini-2.0-flash | Multimodal, long context |
| **Mistral** | 500K tokens/month | 1 RPS | mistral-small | EU compliance |
| **Cohere** | 1,000 req/month | 5 | command-r | RAG + embeddings |
| **Hugging Face** | Not specified | Low | zephyr-7b-beta | Research, variety |
| **NVIDIA NIM** | ~1,000 req (credits) | Varies | llama-3.1-70b | GPU-optimised |
| **Ollama (local)** | Unlimited | Hardware limited | llama3.1:8b | Privacy, offline |

## Strategy for n8n Workflows Hitting Rate Limits

```
1. Add Wait node (2-5 seconds) before the LLM node
2. Enable Retry On Fail in the node's Settings tab
   → Max Tries: 3, Wait Between Tries: 5000ms
3. Use Loop Over Items with small batch sizes (1-5)
4. For large volumes: switch to Ollama (local) or paid tier
5. Monitor quota: set up n8n Error Trigger to alert you when hitting limits
```

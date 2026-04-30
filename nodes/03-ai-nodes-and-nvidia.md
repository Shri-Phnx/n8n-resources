# n8n AI Nodes & NVIDIA NIM Integration — Complete Guide

> **Author:** Shrinivas Ramaprasad  
> **Last updated:** April 2026

---

## Overview

n8n has a native AI framework built on LangChain that enables building production-grade AI pipelines without writing LangChain code. Nodes are grouped into:

- **Root nodes** — orchestrate the AI pipeline (agent, chain, classifier)
- **Sub-nodes** — plug into root nodes (LLM models, memory, tools, vector stores, embeddings)

---

## Root Nodes (AI Orchestrators)

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
1. Receives user input
2. Reasons about which tool(s) to use
3. Calls tools (HTTP Request, Calculator, Workflow, etc.)
4. Incorporates tool results
5. Repeats until it has the final answer
6. Returns final response

**Required sub-nodes:**
- At least one **Chat Model** sub-node
- At least one **Tool** sub-node (optional but makes it useful)
- Optionally a **Memory** sub-node for conversation history

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

### Question and Answer Chain

RAG (Retrieval Augmented Generation) pipeline — answers questions from a document store.

| Parameter | Detail |
|-----------|--------|
| **Query** | User's question |
| **Include source documents** | Return which documents were used to answer |

**Required sub-nodes:** Chat Model + Vector Store Retriever

**Use case:** Chat with PDFs, internal knowledge bases, document Q&A

---

### Summarization Chain

Summarise long text that may exceed context window limits.

| Mode | Detail |
|------|--------|
| **Map Reduce** | Summarise chunks, then summarise summaries |
| **Stuff** | Pass all text at once (for shorter docs) |
| **Refine** | Iteratively refine summary with each chunk |

---

### Information Extractor

Extract structured data fields from unstructured text.

| Parameter | Detail |
|-----------|--------|
| **Schema** | Define fields to extract (name, type, description) |
| **Input text** | Text to extract from (email, document, message) |

**Use case:** Extract invoice fields from emails, parse CVs, structure customer feedback

---

### Text Classifier

Classify text into predefined categories.

| Parameter | Detail |
|-----------|--------|
| **Categories** | List of category names with optional descriptions |
| **Input text** | Text to classify |
| **Allow multiple classes** | Return multiple matching categories |
| **Fallback class** | Class if no match found |

**Use case:** Route support tickets, tag content, categorise emails

---

### Sentiment Analysis

Analyse the sentiment of text.

| Output | Values |
|--------|--------|
| **Sentiment** | Positive, Negative, Neutral |
| **Confidence** | Score from 0 to 1 |

---

## Sub-nodes — LLM Chat Models

### OpenAI Chat Model

| Parameter | Detail |
|-----------|--------|
| **Model** | gpt-4o, gpt-4o-mini, gpt-4-turbo, gpt-3.5-turbo, o1, o3 |
| **Temperature** | 0 (deterministic) to 2 (creative) |
| **Max tokens** | Output length limit |
| **Credentials** | OpenAI API key |

---

### Anthropic Chat Model (Claude)

| Parameter | Detail |
|-----------|--------|
| **Model** | claude-opus-4-6, claude-sonnet-4-6, claude-haiku-4-5 |
| **Temperature** | 0 to 1 |
| **Max tokens** | Output length limit |
| **Credentials** | Anthropic API key |

---

### Google Gemini Chat Model

| Parameter | Detail |
|-----------|--------|
| **Model** | gemini-2.0-flash, gemini-1.5-pro, gemini-1.5-flash |
| **Temperature** | 0 to 1 |
| **Credentials** | Google AI API key or Vertex AI service account |

---

### Ollama Chat Model (Local)

| Parameter | Detail |
|-----------|--------|
| **Model** | llama3.1, mistral, phi3, qwen2.5, deepseek-r1, gemma3 |
| **Base URL** | Local Ollama server (default: http://localhost:11434) |
| **Use case** | Fully local, private, no API cost |

---

### DeepSeek Chat Model

| Parameter | Detail |
|-----------|--------|
| **Model** | deepseek-chat (V3), deepseek-reasoner (R1) |
| **Credentials** | DeepSeek API key |
| **Strength** | Strong reasoning, long context, low cost |

---

### OpenRouter Chat Model

Access 100+ models (OpenAI, Anthropic, Mistral, Meta, NVIDIA) from one API.

| Parameter | Detail |
|-----------|--------|
| **Model** | Any model on openrouter.ai (e.g., `nvidia/llama-3.1-nemotron-70b-instruct`) |
| **Credentials** | OpenRouter API key |
| **Best for** | NVIDIA NIM models, model comparison, fallback routing |

---

### Groq Chat Model

| Parameter | Detail |
|-----------|--------|
| **Model** | llama-3.3-70b-versatile, mixtral-8x7b-32768, gemma2-9b-it |
| **Strength** | Ultra-fast inference (runs on Groq LPU hardware) |
| **Credentials** | Groq API key (free tier available) |

---

## NVIDIA NIM Integration

### What is NVIDIA NIM?

NVIDIA NIM is a set of accelerated inference microservices that allow organisations to run AI models on NVIDIA GPUs anywhere — in the cloud, data centre, workstations, and PCs.

NIM exposes an **OpenAI-compatible API**, making integration with n8n straightforward via multiple methods.

---

### Method 1 — Community Node (Recommended for Chat + Vision)

**Package:** `n8n-nodes-nvidia-nim` by Akash9078

**Install:**
1. Go to **Settings → Community Nodes**
2. Click **Install**
3. Enter package name: `n8n-nodes-nvidia-nim`
4. Agree to risks → click **Install**

**Nodes added:**
- **NVIDIA NIM Chat** — Chat completions with any NIM model
- **NVIDIA NIM Image Analysis** — Vision/multimodal with models like `meta/llama-3.2-11b-vision-instruct`

**Configuration:**
1. Get API key from [build.nvidia.com](https://build.nvidia.com)
2. Create NVIDIA NIM credentials in n8n (API key)
3. Add NVIDIA NIM Chat node → select model → configure prompt

**Available models (examples):**
```
meta/llama-3.1-70b-instruct
meta/llama-3.1-405b-instruct
nvidia/llama-3.1-nemotron-70b-instruct
meta/llama-3.2-11b-vision-instruct
nvidia/nemotron-4-340b-instruct
mistralai/mistral-large-2-instruct
google/gemma-2-27b-it
microsoft/phi-3.5-mini-instruct
```

---

### Method 2 — HTTP Request Node (Direct API Call)

Use when you need full control over the API call.

**Endpoint:** `https://integrate.api.nvidia.com/v1/chat/completions`

**HTTP Request node configuration:**

```
Method: POST
URL: https://integrate.api.nvidia.com/v1/chat/completions

Headers:
  Authorization: Bearer {{ $credentials.nvidiaNimApiKey }}
  Content-Type: application/json

Body (JSON):
{
  "model": "meta/llama-3.1-70b-instruct",
  "messages": [
    {
      "role": "system",
      "content": "You are a helpful IT asset management assistant."
    },
    {
      "role": "user",
      "content": "{{ $json.userMessage }}"
    }
  ],
  "max_tokens": 1024,
  "temperature": 0.7,
  "top_p": 1.0,
  "stream": false
}
```

**Extract the response:**
```
{{ $json.choices[0].message.content }}
```

---

### Method 3 — OpenRouter Chat Model Node

Access NVIDIA NIM models through OpenRouter without a separate NVIDIA account.

**Models available via OpenRouter:**
```
nvidia/llama-3.1-nemotron-70b-instruct
nvidia/nemotron-4-340b-instruct
nvidia/llama-3.3-nemotron-super-49b-v1
```

**Setup:**
1. Create account at [openrouter.ai](https://openrouter.ai)
2. Generate API key
3. In n8n: Add OpenRouter Chat Model sub-node
4. Enter your OpenRouter API key
5. Set model to `nvidia/llama-3.1-nemotron-70b-instruct`

---

### Method 4 — Self-Hosted NIM (Enterprise / On-Premises)

For self-hosted NIM on NVIDIA GPU infrastructure:

**Deploy NIM container:**
```bash
# Pull and run a NIM container (example: Llama 3.1 8B)
docker run -it --rm \
  --gpus all \
  --shm-size=16GB \
  -e NGC_API_KEY=$NGC_API_KEY \
  -v "$LOCAL_NIM_CACHE:/opt/nim/.cache" \
  -u $(id -u) \
  -p 8000:8000 \
  nvcr.io/nim/meta/llama-3.1-8b-instruct:latest
```

**Use in n8n HTTP Request node:**
```
URL: http://your-gpu-server:8000/v1/chat/completions
```
Same OpenAI-compatible payload as Method 2.

---

### NVIDIA NIM — Available Model Tiers

| Tier | Description | Who it's for |
|------|-------------|-------------|
| **NIM Day 0** | Validated within 72 hours of model release; free | Early exploration, all users |
| **NIM Turbo** | Best-in-class inference performance; free in production | Performance-critical applications |
| **NIM Certified** | Enterprise — CVE SLAs, FIPS, long-term support; requires NVIDIA AI Enterprise | Regulated enterprises, ISVs |

---

## Sub-nodes — Memory

| Node | Storage | Persistence | Use Case |
|------|---------|-------------|----------|
| **Simple Memory** | In-memory | Session only | Testing, single-user demos |
| **Redis Chat Memory** | Redis | Persistent | Multi-session, scalable production |
| **Postgres Chat Memory** | PostgreSQL | Persistent | When you already use Postgres |
| **MongoDB Chat Memory** | MongoDB | Persistent | Document-oriented stacks |
| **Chat Memory Manager** | Configurable | Configurable | Advanced memory management |

**Simple Memory configuration:**
- **Session ID** — Groups messages by conversation. Use `{{ $sessionId }}` or a user ID.
- **Context window** — Number of previous messages to include in context.

---

## Sub-nodes — Tools (for AI Agent)

| Tool Node | What the Agent Can Do |
|-----------|----------------------|
| **HTTP Request Tool** | Call any external API |
| **Calculator** | Perform math operations |
| **Wikipedia Tool** | Search and retrieve Wikipedia content |
| **Vector Store Tool** | Semantic search in a vector database |
| **Custom Code Tool** | Run custom JavaScript/Python logic |
| **Call n8n Workflow Tool** | Invoke another n8n workflow as a tool |
| **SerpApi Tool** | Perform Google searches |
| **SearXNG Tool** | Self-hosted search (privacy-preserving) |
| **Think Tool** | Enable chain-of-thought reasoning |
| **AI Agent Tool** | Nest another AI agent as a tool |

---

## Sub-nodes — Vector Stores

| Node | Backend | Notes |
|------|---------|-------|
| **Pinecone Vector Store** | Pinecone cloud | Fully managed, production-ready |
| **PGVector Vector Store** | PostgreSQL + pgvector | Great with existing Postgres stack |
| **Supabase Vector Store** | Supabase | Managed Postgres with vector support |
| **Chroma Vector Store** | Chroma | Open-source, easy local setup |
| **Qdrant Vector Store** | Qdrant | High performance, rich filtering |
| **Weaviate Vector Store** | Weaviate | Hybrid search (vector + keyword) |
| **Redis Vector Store** | Redis | Low latency, in-memory |
| **MongoDB Atlas Vector Store** | MongoDB Atlas | If you're already on MongoDB |
| **Simple Vector Store** | In-memory | Testing only — not persistent |
| **Azure AI Search** | Azure | Enterprise Azure stack |

**Modes:**
- **Insert** — Add documents to the store (build phase)
- **Retrieve** — Search similar documents (query phase)
- **Retrieve as Tool** — Expose to AI Agent

---

## Sub-nodes — Embeddings

| Node | Provider | Model Examples |
|------|---------|----------------|
| **OpenAI Embeddings** | OpenAI | text-embedding-3-small, text-embedding-3-large |
| **Google Gemini Embeddings** | Google | text-embedding-004 |
| **Ollama Embeddings** | Local Ollama | nomic-embed-text, mxbai-embed-large |
| **Hugging Face Embeddings** | HuggingFace | sentence-transformers/all-MiniLM-L6-v2 |
| **Mistral Embeddings** | Mistral | mistral-embed |
| **AWS Bedrock Embeddings** | AWS | titan-embed-text-v2 |
| **Azure OpenAI Embeddings** | Azure | Azure-hosted OpenAI models |

---

## Complete AI Agent Workflow Example

### IT Helpdesk Agent

```
Chat Trigger
    └── AI Agent
          ├── [Model]   Anthropic Chat Model (claude-sonnet-4-6)
          ├── [Memory]  Redis Chat Memory (session: {{ $sessionId }})
          └── [Tools]
                ├── HTTP Request Tool → ServiceNow API
                ├── HTTP Request Tool → JIRA API
                └── Call Workflow Tool → "Send Email Notification"
```

**System prompt:**
```
You are an IT helpdesk assistant for [Company].
You can:
1. Look up IT assets in ServiceNow (use the ServiceNow tool)
2. Create and update JIRA tickets (use the JIRA tool)
3. Send email notifications (use the Email tool)

Always confirm the user's intent before creating or modifying records.
Current date: {{ $now.toFormat('dd MMM yyyy') }}
```

---

## RAG (Retrieval Augmented Generation) Pipeline

### Build Phase (Index Documents)
```
Manual Trigger
    → Read Files / HTTP Request (fetch docs)
    → Default Data Loader
    → Recursive Character Text Splitter
    → Pinecone Vector Store [Insert mode]
          └── OpenAI Embeddings
```

### Query Phase (Answer Questions)
```
Webhook / Chat Trigger
    → Q&A Chain
          ├── [Model] OpenAI Chat Model
          └── [Retriever] Vector Store Retriever
                └── Pinecone Vector Store [Retrieve mode]
                      └── OpenAI Embeddings
    → Respond to Webhook
```

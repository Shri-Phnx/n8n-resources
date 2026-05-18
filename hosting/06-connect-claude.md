# Connect n8n with Claude (Anthropic)

> **Author:** Shrinivas Ramaprasad | **Updated:** May 2026
> How to use Claude as an LLM in n8n, and how to expose n8n as an MCP server for Claude Desktop.

---

## Overview: Three Ways to Connect n8n and Claude

| Method | What it does | Best for |
|---|---|---|
| **Method 1: Claude as LLM in n8n** | Use Claude (Anthropic API) as the AI brain inside n8n AI Agent or LLM Chain | Building AI automation workflows with Claude's intelligence |
| **Method 2: n8n as MCP server for Claude** | Expose n8n workflows as tools that Claude Desktop can call | Letting Claude trigger your automations from chat |
| **Method 3: Claude Desktop → n8n webhook** | Claude sends HTTP requests to n8n webhooks | Simple integration without MCP setup |

---

## Method 1 — Use Claude as LLM Inside n8n Workflows

### WHY: Claude is one of the best LLMs for structured output, reasoning, and following complex instructions — ideal for n8n automation.

### Step 1 — Get Anthropic API Key

```
1. Go to: https://console.anthropic.com
2. Sign in / create account
3. Left sidebar → API Keys → + Create Key
   Name: n8n-automation
4. Copy the key — starts with: sk-ant-api03-...
   ⚠️ Shown only once — save to password manager
```

### Step 2 — Add Anthropic Credential in n8n

```
1. n8n → Settings (gear icon) → Credentials → + Add Credential
2. Search: Anthropic
3. Credential Name: Anthropic - Claude Prod
4. API Key: sk-ant-api03-your-key
5. Save → ✅ Connected
```

### Step 3 — Add Claude to a Workflow

**Option A — AI Agent (Claude as reasoning engine with tools):**

```
1. Add AI Agent node to your workflow
2. Click the Chat Model port (bottom of AI Agent)
3. Search: Anthropic Chat Model → add it
4. Configure Anthropic Chat Model:
   Credential: Anthropic - Claude Prod
   Model: claude-sonnet-4-6  ← best balance of quality/speed/cost
         claude-opus-4-6    ← most capable, higher cost
         claude-haiku-4-5   ← fastest, lowest cost
   Max Tokens: 1024 (for short tasks) to 4096 (for complex outputs)
   Temperature: 0 (deterministic) to 1 (creative). 0.3–0.5 for automation.
5. Add system prompt to AI Agent:
   "You are an IT automation assistant. Today is {{ $now.toFormat('dd MMMM yyyy') }}.
   You help users create support tickets and look up asset information.
   Always confirm before making any changes."
6. Optionally add:
   - Memory sub-node (Redis Chat Memory for persistent conversations)
   - Tool sub-nodes (HTTP Request Tool, Calculator, etc.)
```

**Option B — Basic LLM Chain (Claude for simple text tasks):**

```
1. Add Basic LLM Chain node
2. Add Anthropic Chat Model sub-node
3. Prompt: "Classify this support ticket into one of: Hardware, Software, Network, Access
           Ticket: {{ $json.ticketText }}
           Reply with ONLY the category name."
4. Output: clean category string for routing
```

### Available Claude Models in n8n

| Model ID | Speed | Cost | Context | Best For |
|---|---|---|---|---|
| `claude-haiku-4-5-20251001` | Fastest | Cheapest | 200K | Simple classification, quick tasks |
| `claude-sonnet-4-6` | Balanced | Medium | 200K | Most automation tasks |
| `claude-opus-4-6` | Slowest | Most | 200K | Complex reasoning, analysis |

---

## Method 2 — n8n as MCP Server (Claude Desktop Calls n8n)

### WHY: This lets you type in Claude Desktop "Create an IT ticket for my VPN issue" and Claude automatically calls your n8n workflow to do it.

### How MCP Works with n8n

```
You type in Claude Desktop:
"Search ServiceNow for my open incidents"

Claude recognises this matches the n8n MCP tool "search_servicenow"
          ↓
Claude calls: POST https://your-n8n.com/webhook/mcp-search
{ "query": "open incidents", "user": "shrinivas" }
          ↓
n8n workflow runs (MCP Server Trigger)
          ↓
n8n calls ServiceNow API
          ↓
n8n returns results to Claude
          ↓
Claude shows you the results in chat
```

### Step 1 — Create an n8n MCP Workflow

```
1. Create new workflow: "MCP: Search ServiceNow"
2. Add MCP Server Trigger node (find it under "Other ways")
3. Configure MCP Server Trigger:
   Tool Name: search_servicenow
   Tool Description: Search ServiceNow for incidents, assets, or requests.
                     Input: { query: string, type: "incident"|"asset"|"request" }
                     Returns: list of matching records
4. Add your workflow logic:
   HTTP Request → ServiceNow API
5. The last node's output is what Claude receives
6. Activate the workflow
```

### Step 2 — Get the MCP Endpoint URL

```
With the workflow active, the MCP endpoint is:
https://your-n8n.com/mcp/workflow/{WORKFLOW_ID}/sse

To find WORKFLOW_ID:
→ Open the workflow → look at the URL:
  https://your-n8n.com/workflow/wf_abc123
                                ↑
                          this is the ID
```

### Step 3 — Configure Claude Desktop

```
1. Find Claude Desktop config file:
   macOS: ~/Library/Application Support/Claude/claude_desktop_config.json
   Windows: %APPDATA%\Claude\claude_desktop_config.json

2. Edit the file (create if it doesn't exist):
```

```json
{
  "mcpServers": {
    "n8n-automation": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "https://your-n8n.com/mcp/workflow/wf_abc123/sse"
      ]
    }
  }
}
```

```
3. Save the file
4. Restart Claude Desktop
5. You'll see a hammer icon (🔨) in Claude Desktop
   → Click it → your n8n tools appear
6. Test: type "search servicenow for open incidents"
   → Claude calls your n8n workflow → returns results
```

### Step 4 — Add Authentication to the MCP Endpoint

```
For security, add a header auth to the MCP Server Trigger:

1. In the MCP Server Trigger node → Authentication → Header Auth
2. Create credential:
   Header Name: X-MCP-Secret
   Header Value: generate with: openssl rand -hex 32
3. In Claude Desktop config, add the header:
```

```json
{
  "mcpServers": {
    "n8n-automation": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "https://your-n8n.com/mcp/workflow/wf_abc123/sse",
        "--header",
        "X-MCP-Secret: your-secret-here"
      ]
    }
  }
}
```

### Multiple n8n Tools for Claude Desktop

```json
{
  "mcpServers": {
    "n8n-it-tools": {
      "command": "npx",
      "args": ["mcp-remote", "https://your-n8n.com/mcp/workflow/wf_TICKETING/sse"]
    },
    "n8n-careerforge": {
      "command": "npx",
      "args": ["mcp-remote", "https://your-n8n.com/mcp/workflow/wf_CAREERFORGE/sse"]
    },
    "n8n-reports": {
      "command": "npx",
      "args": ["mcp-remote", "https://your-n8n.com/mcp/workflow/wf_REPORTS/sse"]
    }
  }
}
```

---

## Method 3 — Simple Webhook (No MCP Needed)

### WHY: If you just want Claude to sometimes trigger a workflow, a simple webhook is faster to set up than full MCP.

```
1. In n8n: Add Webhook trigger
   Path: claude-action
   Method: POST
   Auth: Header Auth (X-Claude-Secret: your-secret)
   Activate workflow

2. In Claude: Use the HTTP tool (if available) or tell Claude:
   "Make a POST request to https://your-n8n.com/webhook/claude-action
    with header X-Claude-Secret: your-secret
    and body: { 'action': 'create_ticket', 'description': 'VPN not working' }"

3. Claude Desktop with Fetch MCP can do this directly:
   claude_desktop_config.json → add mcp-fetch server
```

---

## Use Cases: Claude + n8n

| Use Case | How | What Claude Does |
|---|---|---|
| AI support ticket bot | Telegram Trigger → AI Agent (Claude) → ServiceNow | Classifies and routes tickets |
| CV tailoring | Webhook → Claude (Basic LLM Chain) → Google Docs | Rewrites CV for the role |
| Email triage | Gmail Trigger → Claude (classify) → route to folder/CRM | Reads and categorises emails |
| Meeting notes | Form Trigger (paste notes) → Claude (summarise + action items) → Notion | Structures output |
| CareerForge lead qualifier | Form Trigger → Claude (score and categorise) → HubSpot | Assesses lead fit |
| Recruiter outreach | Spreadsheet data → Claude (personalise) → LinkedIn draft | Personalises messages |

---

## Cost Optimisation Tips

```
1. Use Haiku for simple classification (10x cheaper than Sonnet)
2. Use Sonnet for most automation tasks (best value)
3. Use Opus only for complex analysis or high-stakes decisions
4. Set Temperature: 0 for automation (consistent outputs, no wasted tokens)
5. Keep system prompts concise — every token costs
6. Test with Groq free tier first, switch to Claude when logic is proven
7. Monitor usage at console.anthropic.com → Usage tab
```

---

## Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| `401 Unauthorized` | API key wrong or expired | Check console.anthropic.com → regenerate key |
| `529 Overloaded` | Anthropic API busy | Enable Retry On Fail: 3 tries, 10s wait |
| `400 Bad Request` | Max tokens too high | Check model's max output limit |
| MCP tools not showing in Claude Desktop | Config file wrong | Validate JSON syntax, restart Claude Desktop |
| MCP endpoint not reachable | n8n not accessible from internet | Need VPS or ngrok — localhost won't work for Claude Desktop MCP |
| Tool description confuses Claude | Vague description | Be very specific: input format, output format, when to use |

# n8n Webhook Node — Complete Deep Dive

> **Author:** Shrinivas Ramaprasad  
> **Last updated:** April 2026  
> **Source:** [n8n Webhook Docs](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.webhook/)

---

## Overview

The Webhook node lets n8n receive data from any external system over HTTP. It acts as an HTTP server endpoint — when a request hits the URL, the workflow triggers and processes the payload.

**Two roles the Webhook node plays:**
1. **As a trigger** — starts the workflow when a request arrives
2. **Mid-workflow** — used with the Wait node to pause and resume on an external signal (e.g., payment confirmation, approval callback)

---

## Webhook URLs

n8n generates two URLs for every webhook:

| URL Type | When Active | Use For |
|----------|------------|--------|
| **Test URL** | Only when you click "Listen for test event" in the editor | Development and testing |
| **Production URL** | Only when the workflow is **activated** | Live production integrations |

**URL format:**
```
Test:       https://your-n8n-instance.com/webhook-test/{unique-path}
Production: https://your-n8n-instance.com/webhook/{unique-path}
```

> ⚠️ Never use the test URL in production — it only responds when you are actively testing in the editor.

---

## Core Parameters

| Parameter | Options | Description |
|-----------|---------|-------------|
| **HTTP Method** | GET, POST, PUT, PATCH, DELETE, HEAD | The HTTP method the webhook accepts |
| **Path** | Custom string (e.g., `/my-webhook`) | The URL path — auto-generated or custom |
| **Authentication** | None, Basic Auth, Header Auth, JWT | Secure who can call your webhook |
| **Response Mode** | Immediately, Last node, Using Respond to Webhook node | When and how to send the HTTP response |
| **Response Code** | Any HTTP status code | Default status sent back to caller |
| **Response Data** | All items, First item only, No data | What to include in the response body |
| **Binary Property** | Field name | Accept binary/file uploads |

---

## Response Modes Explained

### 1. Immediately (default)
- n8n responds with `200 OK` as soon as the webhook is received
- Workflow processing continues in the background
- **Use when:** Caller doesn't need the result (e.g., Stripe, GitHub webhooks)

### 2. Last Node
- n8n waits for the entire workflow to complete
- Responds with the output of the last node
- **Use when:** Caller expects a response (e.g., a frontend fetching data)
- ⚠️ Caller must wait — can time out for long workflows

### 3. Using Respond to Webhook Node
- You control exactly what response to send using the **Respond to Webhook** node
- Can be placed anywhere in the workflow — not just at the end
- **Use when:** You need custom response bodies, status codes, or headers
- **Best practice for:** Validating data, returning structured responses, early exits

---

## Authentication Options

### None
- Public endpoint — anyone with the URL can trigger it
- Only appropriate for internal/trusted services

### Basic Auth
- Username + password required in the request
- Caller must send: `Authorization: Basic base64(user:pass)`
- Good for server-to-server internal tools

### Header Auth
- Requires a specific header and value (e.g., `X-API-Key: my-secret`)
- Most common for webhook security (Stripe, GitHub, Shopify use this pattern)
- Configure: Header name + Expected value

### JWT
- Validates a JSON Web Token in the `Authorization: Bearer <token>` header
- Most secure — cryptographically verified

---

## Accessing Webhook Data

Once a request arrives, data is available via expressions:

| Expression | Returns |
|------------|--------|
| `{{ $json.body }}` | Full request body |
| `{{ $json.body.fieldName }}` | A specific body field |
| `{{ $json.headers }}` | All request headers |
| `{{ $json.headers['x-api-key'] }}` | A specific header |
| `{{ $json.query }}` | URL query parameters |
| `{{ $json.query.paramName }}` | A specific query param |
| `{{ $json.params }}` | Dynamic path parameters |

**Example — extracting from a Stripe payment webhook:**
```json
{{ $json.body.data.object.amount }}
{{ $json.body.data.object.customer }}
{{ $json.body.type }}
```

---

## Common Patterns & Use Cases

### Pattern 1 — Receive & Acknowledge (Fire-and-Forget)

```
Webhook (Mode: Immediately) → Process Data → Update Database
```
- Stripe sends payment event
- Webhook responds 200 instantly
- n8n processes the event asynchronously

---

### Pattern 2 — Receive & Respond (Synchronous API)

```
Webhook → If (validate) → [True] Respond to Webhook (200 + data)
                        → [False] Respond to Webhook (400 + error)
```
- Frontend calls webhook expecting a result
- n8n validates, processes, and returns structured JSON
- Must use **Response Mode: Using Respond to Webhook Node**

---

### Pattern 3 — Human Approval via Webhook

```
Trigger → Send Email (with approve/reject links)
        → Wait (webhook mode) 
        → If ($json.body.action == 'approve') → Continue / Stop
```
- Workflow pauses until a human clicks a link
- Each link hits a different webhook URL with different payloads
- Ideal for expense approvals, content review, onboarding

---

### Pattern 4 — Validate Webhook Signature (e.g., Stripe)

```
Webhook → Code node (validate HMAC signature) → If (valid) → Process
```

**Stripe signature validation in Code node:**
```javascript
const crypto = require('crypto');
const payload = $json.body;
const signature = $json.headers['stripe-signature'];
const secret = 'whsec_yourStripeWebhookSecret';

const expectedSig = crypto
  .createHmac('sha256', secret)
  .update(JSON.stringify(payload))
  .digest('hex');

if (signature !== `sha256=${expectedSig}`) {
  throw new Error('Invalid Stripe signature');
}

return [{ json: payload }];
```

---

### Pattern 5 — Dynamic Path Parameters

**Webhook path:** `/orders/:orderId/status`

Access the parameter:
```
{{ $json.params.orderId }}
```

Useful for REST-style endpoints where the ID is part of the URL.

---

## Webhook Security Checklist

| Check | Recommendation |
|-------|----------------|
| Authentication | Always use Header Auth or JWT — never leave webhooks open |
| Signature validation | Validate HMAC signatures for services that support it (Stripe, GitHub, Shopify) |
| Rate limiting | Use n8n's IP allowlisting or a reverse proxy (nginx, Cloudflare) |
| HTTPS | Always use HTTPS — never expose webhook over plain HTTP |
| Secret rotation | Rotate webhook secrets regularly |
| Payload size | Validate payload size before processing |
| Idempotency | Handle duplicate events (most services retry on failure) |

---

## Testing Webhooks

### Option 1 — n8n Test Mode
1. Open the Webhook node
2. Click **"Listen for test event"**
3. Send a request using curl, Postman, or your service
4. n8n captures the payload and shows it in the node output

### Option 2 — curl
```bash
# POST with JSON body
curl -X POST https://your-n8n.com/webhook-test/my-path \
  -H "Content-Type: application/json" \
  -H "X-API-Key: your-secret" \
  -d '{"name": "Shrinivas", "event": "test"}'

# GET with query params
curl "https://your-n8n.com/webhook-test/my-path?name=Shrinivas&event=test"
```

### Option 3 — Webhook.site
Use [webhook.site](https://webhook.site) to intercept and inspect webhook payloads from external services before configuring n8n.

### Option 4 — ngrok (local testing)
```bash
# Expose your local n8n to the internet for testing
ngrok http 5678
# Use the generated URL as your webhook endpoint
```

---

## Webhook vs Other Entry Points

| Entry Point | When to Use |
|-------------|-------------|
| **Webhook** | Real-time event from external system (payment, form, API call) |
| **Schedule Trigger** | Regular automated runs at a fixed time |
| **n8n Form Trigger** | When you want n8n to host the form itself |
| **HTTP Request node** | When n8n is the CALLER, not the receiver |
| **Chat Trigger** | When building conversational AI interfaces |

---

## Webhook Configuration for Popular Services

### GitHub
```
Method: POST
Content-Type: application/json
Secret: use Header Auth with X-Hub-Signature-256
Events: Select which repo events to send
```

### Stripe
```
Method: POST
Content-Type: application/json
Validate: stripe-signature header (HMAC SHA256)
Events: payment_intent.succeeded, charge.failed, etc.
```

### Shopify
```
Method: POST
Content-Type: application/json
Validate: X-Shopify-Hmac-SHA256 header
Format: JSON
```

### Slack
```
Method: POST
Challenge response: Must respond to url_verification events
Content-Type: application/json
```

### HubSpot
```
Method: POST
Auth: X-HubSpot-Signature header validation
Events: contact.creation, deal.propertyChange, etc.
```

---

## Respond to Webhook Node — Full Reference

| Parameter | Options | Detail |
|-----------|---------|--------|
| **Respond with** | JSON, Text, Binary, No Data, Redirect | Format of the response body |
| **Response Body** | Any JSON object or expression | The data to return |
| **Response Code** | 200, 201, 400, 401, 403, 404, 500, etc. | HTTP status code |
| **Response Headers** | Key-value pairs | Custom headers (CORS, Content-Type, etc.) |

**Example — returning a structured API response:**
```json
{
  "success": true,
  "message": "Record created",
  "id": "{{ $json.insertedId }}",
  "timestamp": "{{ $now.toISO() }}"
}
```

# n8n Webhook Node — Complete Deep Dive

> **Author:** Shrinivas Ramaprasad | **Last updated:** April 2026
> Every parameter, every option, every field — what it does in n8n, what you must configure in the external service, and how to test from Postman, curl, and other tools.

---

## Overview

The Webhook node makes n8n act as an **HTTP server**. It creates a unique URL that external services can call to trigger your workflow and send data.

**Two distinct roles:**
1. **As a Trigger** — starts the workflow when a request arrives (Webhook Trigger node)
2. **Mid-workflow** — used with the Wait node to pause and resume on an external callback

---

## Test URL vs Production URL — The Most Important Distinction

| URL Type | When It Works | How to Activate | Format |
|---|---|---|---|
| **Test URL** | Only while you click "Listen for test event" in the editor | Click the button in the node panel | `.../webhook-test/{path}` |
| **Production URL** | Only when the workflow toggle is **Active** | Flip the Active toggle in editor header | `.../webhook/{path}` |

⚠️ **This is the #1 cause of webhook confusion.** If you give someone the test URL for production use, it will only work when you're actively listening in the editor.

**Full URL examples:**
```
n8n at https://n8n.yourdomain.com, path = stripe-payment:

Test URL:       https://n8n.yourdomain.com/webhook-test/stripe-payment
Production URL: https://n8n.yourdomain.com/webhook/stripe-payment
```

---

## Parameters Tab — Every Field, Every Option

### Field 1: HTTP Method

**What it is:** The HTTP verb the webhook will accept. Requests using a different method are rejected.

| Option | What It Means | When to Use | Sample Caller Code |
|---|---|---|---|
| **GET** | Read data from n8n | Health checks, simple triggers, URL-based triggers | `curl https://your-n8n.com/webhook/path` |
| **POST** | Send data to n8n (most common) | Stripe, GitHub, Slack, forms, apps that POST JSON | `curl -X POST -d '{...}' https://...` |
| **PUT** | Full resource update | REST APIs that update a full resource | `curl -X PUT -d '{...}' https://...` |
| **PATCH** | Partial update | APIs that send partial updates | `curl -X PATCH -d '{...}' https://...` |
| **DELETE** | Deletion event | Resource deletion notifications | `curl -X DELETE https://...` |
| **HEAD** | Check if webhook is alive | Ping/health checks | `curl -I https://...` |

**What to do in n8n:** Select the method from the dropdown.

**What to do in the external service:** Set the same method in the service's webhook configuration. For example, in Stripe → set "HTTP method: POST".

**⚠️ If wrong:** The webhook returns `404 Not Found` to the caller. Your workflow never triggers.

**✅ Tip:** When unsure, use `POST` — it accepts a JSON body which carries the most data.

**✅ Tip:** You can accept multiple methods by creating two webhook nodes with the same path but different methods and merging their outputs with a Merge node.

---

### Field 2: Path

**What it is:** The URL path segment that identifies this webhook uniquely. Becomes part of the URL.

| Option | Sample Value | Resulting URL | When to Use |
|---|---|---|---|
| **Auto-generated UUID** | *(leave blank)* | `.../webhook/a3f8-b291-c4d5` | Maximum randomness, hard to guess |
| **Custom path** | `stripe-payment` | `.../webhook/stripe-payment` | Human-readable, easy to manage |
| **Nested path** | `forms/it-support` | `.../webhook/forms/it-support` | Organize multiple webhooks by category |
| **Dynamic path** | `orders/:orderId` | `.../webhook/orders/ORD-001` | REST-style URLs with parameters |

**What to do in n8n:** Type your preferred path. No leading slash.

**What to do in external service:** Use the full webhook URL (from the node's URL panel) in the service's webhook settings.

**⚠️ Common mistake:** Two webhook nodes with the same path will conflict — n8n will only use the first one. Use unique paths per webhook.

**✅ Tip:** Use descriptive paths: `stripe-payment-success`, `github-push-main`, `slack-mention` — not generic names like `test` or `webhook1`.

**Accessing dynamic path parameters:**
```
Path: orders/:orderId/status/:status
Incoming URL: /webhook/orders/ORD-001/status/shipped
Access: {{ $json.params.orderId }}  → ORD-001
        {{ $json.params.status }}   → shipped
```

---

### Field 3: Authentication

**What it is:** Validates that the caller is who they say they are. Prevents unauthorized access to your webhook.

#### Option A: None
| What | Anyone with the URL can trigger your workflow |
|---|---|
| **When to use** | Internal tools on a private network only |
| **Never use for** | Any public-facing webhook |
| **What to do in n8n** | Select "None" |
| **What to do in service** | Nothing — no auth headers needed |
| **Risk** | If someone discovers your URL, they can spam your workflow |

#### Option B: Basic Auth
| What | Username + password required, sent as base64 encoded header |
|---|---|
| **When to use** | Server-to-server calls where Basic Auth is supported |
| **What to do in n8n** | Select "Basic Auth" → Create credential → set Username + Password |
| **What to do in service** | Configure: Username = your chosen username, Password = your password |
| **How caller sends it** | Header: `Authorization: Basic base64(username:password)` |
| **Test with curl** | `curl -u admin:password https://your-n8n.com/webhook/path` |
| **⚠️ Limitation** | Credentials sent in every request — use HTTPS always |

**Creating Basic Auth credential in n8n:**
```
Webhook node → Authentication → Basic Auth
→ Click "Select Credential" → Create New
→ Credential Name: Webhook Basic Auth
→ User: admin
→ Password: (generate: openssl rand -base64 16)
→ Save
```

#### Option C: Header Auth
| What | A secret value sent in a specific HTTP header |
|---|---|
| **When to use** | Most service webhooks (Stripe, GitHub, Shopify use this pattern) |
| **What to do in n8n** | Select "Header Auth" → Create credential → set Header Name + Header Value |
| **What to do in service** | Copy the header name and value into the service's webhook secret field |
| **How caller sends it** | Header: `X-Webhook-Secret: your-secret-value` |
| **Sample Header Name** | `X-Webhook-Secret`, `X-API-Key`, `Authorization` |
| **Sample Header Value** | `sk-abc123...` (generate: `openssl rand -hex 32`) |
| **Test with curl** | `curl -H "X-Webhook-Secret: your-secret" -X POST https://your-n8n.com/webhook/path -d '{"test":true}'` |

**Creating Header Auth credential in n8n:**
```
Webhook node → Authentication → Header Auth
→ Click "Select Credential" → Create New
→ Credential Name: Stripe Webhook Secret
→ Header Name: stripe-signature   ← exactly what Stripe sends
→ Header Value: whsec_your_stripe_secret
→ Save
```

**⚠️ Tip:** Different services use different header names. GitHub uses `X-Hub-Signature-256`. Stripe uses `stripe-signature`. Always check the service's docs.

#### Option D: JWT (JSON Web Token)
| What | Cryptographically signed token verified by n8n |
|---|---|
| **When to use** | High-security scenarios; when callers can generate JWTs |
| **What to do in n8n** | Select "JWT" → Create credential → set Secret Key (symmetric) or Public Key (asymmetric) |
| **What to do in service** | Generate a JWT signed with the matching secret/private key; send in `Authorization: Bearer {token}` header |
| **Algorithms supported** | HS256, HS384, HS512, RS256, RS384, RS512, ES256, ES384, ES512 |
| **Sample header** | `Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...` |

---

### Field 4: Respond (Response Mode)

**What it is:** When and how n8n sends the HTTP response back to the caller.

#### Option A: Immediately (Default)
| What | n8n sends `200 OK` as soon as the webhook request is received. Workflow continues in the background. |
|---|---|
| **What caller receives** | `200 OK` with empty or minimal body, immediately |
| **n8n behaviour** | Workflow runs asynchronously after responding |
| **What to do in n8n** | Select "Immediately" in Respond dropdown |
| **What to do in service** | No special config needed — service just expects a fast 200 response |
| **✅ Use when** | Stripe, GitHub, Shopify — services that don't need response data from n8n |
| **⚠️ Don't use when** | You need to send the workflow result back to the caller |
| **Sample response body** | `{}` or empty |

#### Option B: Last Node
| What | n8n waits for the entire workflow to complete, then responds with the last node's output |
|---|---|
| **What caller receives** | JSON of the last node's output — after all processing is done |
| **n8n behaviour** | Caller connection stays open until workflow finishes |
| **What to do in n8n** | Select "Last Node". The last node's output IS the response |
| **What to do in service** | Set a long enough timeout (at least 30s; some workflows take minutes) |
| **✅ Use when** | Browser/app fetching data from n8n and waiting for result |
| **⚠️ Timeout risk** | If workflow takes >30s, many HTTP clients time out. Use "Respond to Webhook Node" instead for better control |
| **Sample response** | Whatever the last node returns, e.g.: `[{"result": "success", "id": 123}]` |

#### Option C: Using Respond to Webhook Node
| What | You explicitly control when and what to respond using the "Respond to Webhook" node placed anywhere in the workflow |
|---|---|
| **What caller receives** | Exactly what you define in the Respond to Webhook node |
| **n8n behaviour** | Workflow pauses at the Respond node to send the response, then continues |
| **What to do in n8n** | Select this option. Add a "Respond to Webhook" node in your workflow. Configure its body, status code, and headers |
| **What to do in service** | Nothing special — just make sure timeout is appropriate |
| **✅ Use when** | Need custom HTTP status codes (400 for validation error), custom response body, or need to respond early in the workflow |

**Respond to Webhook node configuration:**
```
Respond with: JSON
Response Body: {
  "success": true,
  "message": "Order received",
  "orderId": "{{ $json.orderId }}",
  "timestamp": "{{ $now.toISO() }}"
}
Response Code: 200
Response Headers:
  Content-Type: application/json
  X-Processed-By: n8n
```

---

### Field 5: Response Code

**What it is:** The HTTP status code sent back to the caller.

| Code | Meaning | When to Use |
|---|---|---|
| **200** | OK — success | General success |
| **201** | Created | New resource created |
| **202** | Accepted | Request received, will process async |
| **204** | No Content | Success, no body to return |
| **400** | Bad Request | Invalid input from caller |
| **401** | Unauthorized | Auth failed |
| **403** | Forbidden | Authenticated but not authorized |
| **404** | Not Found | Resource doesn't exist |
| **409** | Conflict | Duplicate detected |
| **422** | Unprocessable | Valid JSON but fails validation |
| **429** | Too Many Requests | Rate limit hit |
| **500** | Internal Error | Something went wrong on n8n side |

**What to do in n8n:** Set appropriate code. For Immediately mode, use 200. For Respond to Webhook node mode, set the code in that node.

**What to do in service:** Most services accept any 2xx code as success. Check service docs for expected codes.

---

### Field 6: Response Data

**What it is:** What to include in the HTTP response body. Only relevant when Respond = "Last Node".

| Option | What It Returns | When to Use |
|---|---|---|
| **All Entries** | All items from the last node as a JSON array | When caller expects an array |
| **First Entry Only** | Just the first item | When caller expects a single object |
| **No Data** | Empty body | When caller doesn't need response data |

---

### Field 7: Binary Property (Advanced)

**What it is:** The field name to use when the incoming request sends binary data (file uploads).

| Sample Value | When to Use |
|---|---|
| `data` | Default — if a file is uploaded via form-data |
| `file` | When the calling service names the field differently |

**What to do in n8n:** Set the field name that matches what the caller sends.

**Accessing the binary data:**
```
Next node → use "Convert to File" or process with Code node:
const binaryData = $binary.data;
```

---

## Options (Additional Settings)

### Option: Ignore Bots
| What | Blocks requests from known bots and crawlers |
|---|---|
| **When to enable** | If your webhook URL is public and you're getting crawler traffic |
| **Checks** | User-Agent header for bot signatures |

### Option: Raw Body
| What | Access the raw unparsed request body as a string |
|---|---|
| **When to use** | When validating HMAC signatures (Stripe, GitHub) — you need the raw body for accurate signature computation |
| **Access** | `{{ $json.rawBody }}` |
| **⚠️ Important** | For Stripe signature validation, you MUST use the raw body — parsed JSON won't match the signature |

### Option: IP Allowlist
| What | Only accept requests from specific IP addresses |
|---|---|
| **Sample value** | `192.168.1.0/24, 10.0.0.5` |
| **When to use** | When the calling service publishes its IP ranges (Stripe, GitHub both publish IP lists) |

---

## Testing Webhooks — Every Method

### Method 1: n8n Test Mode
```
1. Open the Webhook node
2. Click "Listen for test event" button
3. Node shows a spinning indicator: "Listening..."
4. Send a request from Postman/curl/service
5. n8n captures it and shows the data in the node output
6. Click "Stop listening" when done

⚠️ Test URL only works while you're actively listening.
⚠️ If you navigate away, listening stops.
```

### Method 2: curl (Command Line)
```bash
# Basic GET request
curl "https://your-n8n.com/webhook-test/my-path"

# POST with JSON body
curl -X POST https://your-n8n.com/webhook-test/my-path \
  -H "Content-Type: application/json" \
  -H "X-Webhook-Secret: your-secret" \
  -d '{"event": "payment_success", "amount": 5000, "currency": "usd"}'

# POST with Basic Auth
curl -X POST https://your-n8n.com/webhook-test/my-path \
  -u admin:yourpassword \
  -H "Content-Type: application/json" \
  -d '{"name": "Shrinivas"}'

# POST with file upload (multipart)
curl -X POST https://your-n8n.com/webhook-test/my-path \
  -H "X-Webhook-Secret: secret" \
  -F "name=Shrinivas" \
  -F "file=@/path/to/invoice.pdf"

# PUT request
curl -X PUT https://your-n8n.com/webhook-test/my-path \
  -H "Content-Type: application/json" \
  -d '{"id": 123, "status": "active"}'

# View full response headers
curl -v -X POST https://your-n8n.com/webhook-test/my-path \
  -H "Content-Type: application/json" \
  -d '{"test": true}'
```

### Method 3: Postman — Step-by-Step

#### Setup in Postman:
```
Step 1: Open Postman → New Request

Step 2: Set Method
  → Click the method dropdown (GET by default)
  → Select POST (or whatever your webhook accepts)

Step 3: Enter URL
  → Paste the Test URL from n8n:
    https://your-n8n.com/webhook-test/my-path

Step 4: Set Headers
  → Click "Headers" tab
  → Add headers:
    Key: Content-Type       | Value: application/json
    Key: X-Webhook-Secret   | Value: your-secret-here
  → If Basic Auth: skip headers, use Authorization tab instead

Step 5: Set Body (for POST/PUT/PATCH)
  → Click "Body" tab
  → Select "raw"
  → Select "JSON" from the dropdown (right side)
  → Enter your JSON:
    {
      "event": "test_event",
      "name": "Shrinivas Ramaprasad",
      "email": "shrinivas@example.com",
      "amount": 5000,
      "timestamp": "2026-04-30T09:00:00Z"
    }

Step 6: Set Auth (if using Basic Auth)
  → Click "Authorization" tab
  → Type: Basic Auth
  → Username: admin
  → Password: your-password

Step 7: Click "Send"
  → n8n must be in "Listen for test event" mode
  → Postman shows response: 200 OK {}
  → n8n shows the captured data in the node output
```

#### Postman Environment Variables (for switching between test and prod):
```
Create Postman Environment:
  → Click Environments → New
  → Add variables:
    Name: base_url
    Initial Value: https://your-n8n.com
    Current Value: https://your-n8n.com

    Name: webhook_secret
    Initial Value: your-webhook-secret
    Current Value: your-webhook-secret

In your request URL: {{base_url}}/webhook-test/my-path
In headers: X-Webhook-Secret: {{webhook_secret}}
```

#### Postman Collection for n8n Webhooks:
```
Organize all your webhook tests:
→ New Collection → "n8n Webhook Tests"
→ Add requests for each webhook:
  → POST stripe-payment-test
  → POST github-push-test
  → POST slack-mention-test

Save test data as Postman Examples:
→ In each request → click Save as Example
→ Saves your sample payload for team reference
```

### Method 4: Webhook.site (Inspect Before Configuring)
```
1. Go to https://webhook.site
2. Copy the unique URL assigned to you
3. Set this URL temporarily in your external service (Stripe, GitHub, etc.)
4. Trigger an event in the service
5. Webhook.site shows the EXACT request: headers, body, method
6. Now you know exactly what to expect in n8n
7. Replace the webhook.site URL with your n8n URL
```

### Method 5: ngrok (Expose Local n8n to Internet)
```bash
# Install ngrok: ngrok.com/download
ngrok config add-authtoken your-ngrok-token

# Expose local n8n
ngrok http 5678

# Output:
# Forwarding  https://abc123.ngrok-free.app → http://localhost:5678

# Your webhook URL becomes:
https://abc123.ngrok-free.app/webhook-test/my-path

# Set in n8n .env:
WEBHOOK_URL=https://abc123.ngrok-free.app
```

---

## Accessing Webhook Data in Downstream Nodes

### All Available Data
```javascript
// Headers
{{ $json.headers['content-type'] }}         // application/json
{{ $json.headers['x-webhook-secret'] }}      // your-secret
{{ $json.headers['user-agent'] }}            // Stripe/1.0
{{ $json.headers.authorization }}            // Bearer token...

// Body fields
{{ $json.body.fieldName }}                   // any field in POST body
{{ $json.body.nested.value }}                // nested object access
{{ $json.body.items[0].id }}                 // array access

// Query parameters (GET: ?name=value)
{{ $json.query.name }}                       // URL query parameter
{{ $json.query.source }}                     // e.g., 'stripe'

// Path parameters (dynamic paths: /orders/:id)
{{ $json.params.id }}                        // URL path parameter

// Raw body (for signature verification)
{{ $json.rawBody }}                          // unparsed string body

// Request metadata
{{ $json.webhookId }}                        // n8n's webhook ID
```

---

## Webhook Configuration for Popular Services

### Stripe
```
In Stripe Dashboard:
1. Developers → Webhooks → Add Endpoint
2. Endpoint URL: https://n8n.yourdomain.com/webhook/stripe-payment
3. Events to send: payment_intent.succeeded, charge.failed, customer.subscription.created
4. Click Add Endpoint
5. Copy the Signing Secret (starts with whsec_)

In n8n:
- HTTP Method: POST
- Path: stripe-payment
- Authentication: None (validate signature manually in Code node)
- Enable: Raw Body option

Signature validation Code node:
```javascript
const crypto = require('crypto');
const rawBody = $json.rawBody;
const sigHeader = $json.headers['stripe-signature'];
const secret = 'whsec_your_stripe_secret';

// Parse the timestamp and signature from the header
const elements = sigHeader.split(',');
const timestamp = elements.find(e => e.startsWith('t=')).split('=')[1];
const sig = elements.find(e => e.startsWith('v1=')).split('=')[1];

// Create expected signature
const signedPayload = `${timestamp}.${rawBody}`;
const expectedSig = crypto
  .createHmac('sha256', secret)
  .update(signedPayload)
  .digest('hex');

if (sig !== expectedSig) {
  throw new Error('Invalid Stripe signature');
}

return [{ json: JSON.parse(rawBody) }];
```

---

### GitHub
```
In GitHub:
1. Repo → Settings → Webhooks → Add Webhook
2. Payload URL: https://n8n.yourdomain.com/webhook/github-push
3. Content type: application/json
4. Secret: generate with openssl rand -hex 32, paste here
5. Events: select which events (Just push, or let me select: push, pull_request, issues)
6. Active: ✅ checked
7. Add Webhook

In n8n:
- HTTP Method: POST
- Path: github-push
- Authentication: Header Auth
  - Header Name: x-hub-signature-256   (lowercase!)
  - Header Value: sha256=your-hmac-signature
  Note: n8n validates this automatically when you use Header Auth with
        the header name x-hub-signature-256

Useful data expressions:
{{ $json.body.ref }}                          → refs/heads/main
{{ $json.body.repository.name }}              → repo name
{{ $json.body.commits[0].message }}           → commit message
{{ $json.body.pusher.name }}                  → who pushed
{{ $json.body.pull_request.title }}           → PR title
```

---

### Slack
```
In Slack API:
1. api.slack.com/apps → your app
2. Event Subscriptions → Enable Events
3. Request URL: paste your n8n webhook URL
   → Slack sends a "challenge" request first
   → n8n must respond with the challenge value
4. Add Bot Events: message.channels, app_mention, reaction_added
5. Save Changes → Reinstall app

Challenge handling in n8n:
- Respond mode: Using Respond to Webhook Node
- Add If node: if $json.body.type == 'url_verification'
    → Respond to Webhook: { "challenge": "{{ $json.body.challenge }}" }
  else:
    → Continue with real event processing

Useful data:
{{ $json.body.event.type }}                  → message, reaction_added
{{ $json.body.event.text }}                  → message text
{{ $json.body.event.user }}                  → user ID
{{ $json.body.event.channel }}               → channel ID
```

---

### HubSpot
```
In HubSpot:
1. Settings → Integrations → Private Apps → Create Private App
2. Scopes: contacts, deals, companies
3. Create → copy the token
4. Settings → Notifications/Webhooks
5. Target URL: https://n8n.yourdomain.com/webhook/hubspot
6. Subscribe to: contact.creation, deal.propertyChange
7. Save

In n8n:
- HTTP Method: POST
- Path: hubspot
- Authentication: Header Auth
  - Header Name: X-HubSpot-Signature
  - Header Value: (auto-computed by HubSpot)

Useful data:
{{ $json.body[0].subscriptionType }}         → contact.creation
{{ $json.body[0].objectId }}                 → contact ID
{{ $json.body[0].propertyName }}             → which property changed
{{ $json.body[0].propertyValue }}            → new value
```

---

### Shopify
```
In Shopify:
1. Admin → Settings → Notifications → Webhooks
2. Create Webhook:
   - Event: Order payment (or others)
   - Format: JSON
   - URL: https://n8n.yourdomain.com/webhook/shopify-order
   - Webhook API Version: latest
3. Save → copy the signing secret shown

In n8n:
- HTTP Method: POST
- Path: shopify-order
- Authentication: None (validate with HMAC)
- Enable: Raw Body

HMAC validation:
const hash = crypto.createHmac('sha256', 'your_shopify_secret')
  .update($json.rawBody, 'utf8')
  .digest('base64');
const shopifyHmac = $json.headers['x-shopify-hmac-sha256'];
if (hash !== shopifyHmac) throw new Error('Invalid Shopify signature');

Useful data:
{{ $json.body.id }}                          → order ID
{{ $json.body.email }}                       → customer email
{{ $json.body.total_price }}                 → order total
{{ $json.body.financial_status }}            → paid, pending
```

---

### Postman (Testing n8n Webhooks from Postman)
```
Using Postman as the webhook caller/tester:

1. Open Postman → New Collection → "n8n Webhook Tests"

2. Add Request:
   Name: Test Stripe Webhook
   Method: POST
   URL: https://your-n8n.com/webhook-test/stripe-payment

3. Headers tab:
   Content-Type: application/json
   stripe-signature: t=1714456800,v1=abc123def456...
   (or X-Webhook-Secret for Header Auth)

4. Body tab → raw → JSON:
   {
     "id": "evt_test_001",
     "type": "payment_intent.succeeded",
     "data": {
       "object": {
         "id": "pi_test_001",
         "amount": 5000,
         "currency": "usd",
         "customer": "cus_test_001",
         "receipt_email": "shrinivas@example.com",
         "status": "succeeded"
       }
     },
     "created": 1714456800
   }

5. Pre-request Script (to simulate Stripe signature):
   const secret = 'whsec_test_secret';
   const timestamp = Math.floor(Date.now() / 1000);
   const body = pm.request.body.raw;
   const payload = timestamp + '.' + body;
   const signature = CryptoJS.HmacSHA256(payload, secret).toString();
   pm.request.headers.upsert({
     key: 'stripe-signature',
     value: `t=${timestamp},v1=${signature}`
   });

6. Save as Example for documentation

7. Add to Postman Monitor (optional):
   → Run tests on a schedule to verify webhook health
```

---

## Common Webhook Mistakes & Solutions

| Mistake | Effect | Solution |
|---|---|---|
| Using test URL in production | Webhook only fires when editor is open | Use production URL (no `-test`); activate workflow |
| Workflow not active | Production webhook returns 404 | Toggle Active switch in editor |
| WEBHOOK_URL set to localhost | External services can't reach your n8n | Set WEBHOOK_URL=https://your-public-domain.com |
| Wrong HTTP method configured | Caller gets 404 | Match method to what service sends (usually POST) |
| No signature validation | Anyone can trigger your workflow | Add Header Auth or validate HMAC in Code node |
| Parsing signed body after JSON parse | Signature validation fails | Enable Raw Body option in webhook |
| Response timeout | Service retries and sends duplicate events | Use Respond mode: Immediately for fire-and-forget |
| Duplicate processing | Service retries on timeout | Add idempotency check using Redis or Postgres |
| SSL error | Certificate not valid | Ensure HTTPS with valid cert (Let's Encrypt) |
| Port 5678 exposed directly | No SSL, no reverse proxy | Use nginx reverse proxy (see hosting guide) |

---

## Security Checklist

```
⬜ Use HTTPS — never expose webhook over HTTP
⬜ Add authentication (Header Auth minimum)
⬜ Validate HMAC signatures for services that support it
⬜ Use allowlist for known service IPs where possible
⬜ Rotate secrets every 90 days
⬜ Use Immediately response mode for external services
⬜ Add idempotency for critical workflows
⬜ Log all webhook events for audit trail
⬜ Set up Error Trigger to alert on webhook failures
⬜ Test with Postman before going live
```
